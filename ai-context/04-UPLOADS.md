# File Uploads Module - AI Context

## 📁 **File Upload & Storage Patterns**

Load this context when working with file upload features.

---

## 🏗️ **Upload Architecture**

### **Upload Flow**
```
Client → Upload API → Storage Provider (S3/Cloudinary) → Database Record → CDN → User
```

### **Key Components**
- **Upload API**: Handles file validation and upload initiation
- **Storage Service**: Abstracts storage provider operations
- **File Processing**: Image resizing, video transcoding, etc.
- **CDN Integration**: Fast file delivery

---

## 🗂️ **Storage Configuration**

### **AWS S3 Setup**
```typescript
// lib/storage.ts
import { S3Client } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import { PutObjectCommand, DeleteObjectCommand } from '@aws-sdk/client-s3';

export const s3Client = new S3Client({
  region: process.env.AWS_REGION!,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

export const STORAGE_CONFIG = {
  bucket: process.env.AWS_S3_BUCKET!,
  region: process.env.AWS_REGION!,
  cdnUrl: process.env.CDN_URL, // Optional CloudFront URL
  maxFileSize: 10 * 1024 * 1024, // 10MB
  allowedTypes: [
    'image/jpeg',
    'image/png', 
    'image/gif',
    'image/webp',
    'application/pdf',
    'text/csv',
  ],
} as const;
```

### **Cloudinary Setup (Alternative)**
```typescript
// lib/cloudinary.ts
import { v2 as cloudinary } from 'cloudinary';

cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET,
});

export { cloudinary };
```

---

## 📤 **Upload Service Patterns**

### **Core Upload Service**
```typescript
// services/upload.service.ts
export class UploadService {
  /**
   * Generates presigned URL for direct S3 upload
   */
  static async generatePresignedUrl({
    userId,
    fileName,
    fileType,
    fileSize,
  }: {
    userId: string;
    fileName: string;
    fileType: string;
    fileSize: number;
  }): Promise<{ uploadUrl: string; fileKey: string }> {
    // Validate file type
    if (!STORAGE_CONFIG.allowedTypes.includes(fileType as any)) {
      throw new Error('File type not allowed');
    }

    // Validate file size
    if (fileSize > STORAGE_CONFIG.maxFileSize) {
      throw new Error('File too large');
    }

    // Generate unique file key
    const timestamp = Date.now();
    const random = Math.random().toString(36).substring(7);
    const extension = fileName.split('.').pop();
    const fileKey = `uploads/${userId}/${timestamp}-${random}.${extension}`;

    // Create presigned URL
    const command = new PutObjectCommand({
      Bucket: STORAGE_CONFIG.bucket,
      Key: fileKey,
      ContentType: fileType,
      ContentLength: fileSize,
    });

    const uploadUrl = await getSignedUrl(s3Client, command, {
      expiresIn: 3600, // 1 hour
    });

    return { uploadUrl, fileKey };
  }

  /**
   * Records file upload in database
   */
  static async recordUpload({
    userId,
    fileName,
    fileKey,
    fileType,
    fileSize,
    purpose = 'general',
  }: {
    userId: string;
    fileName: string;
    fileKey: string;
    fileType: string;
    fileSize: number;
    purpose?: string;
  }): Promise<Upload> {
    const publicUrl = STORAGE_CONFIG.cdnUrl 
      ? `${STORAGE_CONFIG.cdnUrl}/${fileKey}`
      : `https://${STORAGE_CONFIG.bucket}.s3.${STORAGE_CONFIG.region}.amazonaws.com/${fileKey}`;

    return await db.upload.create({
      data: {
        userId,
        fileName,
        fileKey,
        fileType,
        fileSize,
        purpose,
        url: publicUrl,
        status: 'COMPLETED',
      },
    });
  }

  /**
   * Deletes file from storage and database
   */
  static async deleteFile(uploadId: string, userId: string): Promise<void> {
    // Get upload record
    const upload = await db.upload.findFirst({
      where: { id: uploadId, userId }, // Security: only delete own files
    });

    if (!upload) {
      throw new Error('File not found');
    }

    // Delete from S3
    await s3Client.send(new DeleteObjectCommand({
      Bucket: STORAGE_CONFIG.bucket,
      Key: upload.fileKey,
    }));

    // Delete from database
    await db.upload.delete({
      where: { id: uploadId },
    });
  }

  /**
   * Gets user uploads with pagination
   */
  static async getUserUploads(
    userId: string,
    { page = 1, limit = 20, purpose }: { page?: number; limit?: number; purpose?: string } = {}
  ): Promise<{ uploads: Upload[]; total: number }> {
    const where = { userId, ...(purpose && { purpose }) };

    const [uploads, total] = await Promise.all([
      db.upload.findMany({
        where,
        orderBy: { createdAt: 'desc' },
        skip: (page - 1) * limit,
        take: limit,
      }),
      db.upload.count({ where }),
    ]);

    return { uploads, total };
  }
}
```

### **Image Processing Service**
```typescript
// services/image-processing.service.ts
import sharp from 'sharp';

export class ImageProcessingService {
  /**
   * Processes uploaded images (resize, optimize)
   */
  static async processImage({
    uploadId,
    variants = ['thumbnail', 'medium', 'large'],
  }: {
    uploadId: string;
    variants?: string[];
  }): Promise<void> {
    const upload = await db.upload.findUnique({
      where: { id: uploadId },
    });

    if (!upload || !upload.fileType.startsWith('image/')) {
      return;
    }

    // Download original from S3
    const response = await fetch(upload.url);
    const buffer = Buffer.from(await response.arrayBuffer());

    // Process variants
    const variantPromises = variants.map(async (variant) => {
      const dimensions = this.getVariantDimensions(variant);
      
      const processedBuffer = await sharp(buffer)
        .resize(dimensions.width, dimensions.height, {
          fit: 'inside',
          withoutEnlargement: true,
        })
        .jpeg({ quality: 85, progressive: true })
        .toBuffer();

      // Upload processed image
      const variantKey = upload.fileKey.replace('.', `_${variant}.`);
      
      await s3Client.send(new PutObjectCommand({
        Bucket: STORAGE_CONFIG.bucket,
        Key: variantKey,
        Body: processedBuffer,
        ContentType: 'image/jpeg',
      }));

      // Create variant URL
      const variantUrl = STORAGE_CONFIG.cdnUrl 
        ? `${STORAGE_CONFIG.cdnUrl}/${variantKey}`
        : `https://${STORAGE_CONFIG.bucket}.s3.${STORAGE_CONFIG.region}.amazonaws.com/${variantKey}`;

      // Save variant to database
      await db.uploadVariant.create({
        data: {
          uploadId,
          variant,
          url: variantUrl,
          fileKey: variantKey,
          width: dimensions.width,
          height: dimensions.height,
        },
      });
    });

    await Promise.all(variantPromises);

    // Update upload status
    await db.upload.update({
      where: { id: uploadId },
      data: { status: 'PROCESSED' },
    });
  }

  private static getVariantDimensions(variant: string): { width: number; height: number } {
    const dimensions = {
      thumbnail: { width: 150, height: 150 },
      medium: { width: 500, height: 500 },
      large: { width: 1200, height: 1200 },
    };

    return dimensions[variant as keyof typeof dimensions] || { width: 800, height: 800 };
  }
}
```

---

## 📡 **Upload API Routes**

### **Presigned URL Generation**
```typescript
// app/api/uploads/presign/route.ts
export async function POST(request: NextRequest) {
  try {
    // Check authentication
    const session = await auth();
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const body = await request.json();
    const { fileName, fileType, fileSize, purpose } = body;

    // Generate presigned URL
    const { uploadUrl, fileKey } = await UploadService.generatePresignedUrl({
      userId: session.user.id,
      fileName,
      fileType,
      fileSize,
    });

    return NextResponse.json({
      uploadUrl,
      fileKey,
      expiresIn: 3600,
    });
  } catch (error) {
    console.error('Presign error:', error);
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Upload failed' },
      { status: 400 }
    );
  }
}
```

### **Upload Completion Handler**
```typescript
// app/api/uploads/complete/route.ts
export async function POST(request: NextRequest) {
  try {
    const session = await auth();
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const body = await request.json();
    const { fileName, fileKey, fileType, fileSize, purpose } = body;

    // Record upload in database
    const upload = await UploadService.recordUpload({
      userId: session.user.id,
      fileName,
      fileKey,
      fileType,
      fileSize,
      purpose,
    });

    // Process image if needed
    if (fileType.startsWith('image/')) {
      // Queue image processing (async)
      await ImageProcessingService.processImage({
        uploadId: upload.id,
      });
    }

    return NextResponse.json({ upload });
  } catch (error) {
    console.error('Upload completion error:', error);
    return NextResponse.json(
      { error: 'Failed to complete upload' },
      { status: 500 }
    );
  }
}
```

---

## 🎨 **Upload UI Components**

### **File Upload Component**
```typescript
// components/uploads/file-upload.tsx
'use client';

interface FileUploadProps {
  onUploadComplete?: (upload: Upload) => void;
  accept?: string[];
  maxSize?: number;
  purpose?: string;
  multiple?: boolean;
}

export function FileUpload({
  onUploadComplete,
  accept = ['image/*'],
  maxSize = 10 * 1024 * 1024, // 10MB
  purpose = 'general',
  multiple = false,
}: FileUploadProps) {
  const [uploading, setUploading] = useState(false);
  const [progress, setProgress] = useState(0);
  const fileInputRef = useRef<HTMLInputElement>(null);

  const handleFileSelect = async (files: FileList) => {
    if (!files.length) return;

    const filesToUpload = Array.from(files);
    
    // Validate files
    for (const file of filesToUpload) {
      if (file.size > maxSize) {
        toast.error(`File "${file.name}" is too large`);
        return;
      }
    }

    setUploading(true);

    try {
      for (const file of filesToUpload) {
        await uploadFile(file);
      }
    } catch (error) {
      toast.error('Upload failed');
    } finally {
      setUploading(false);
      setProgress(0);
    }
  };

  const uploadFile = async (file: File) => {
    // Get presigned URL
    const response = await fetch('/api/uploads/presign', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        fileName: file.name,
        fileType: file.type,
        fileSize: file.size,
        purpose,
      }),
    });

    if (!response.ok) {
      throw new Error('Failed to get upload URL');
    }

    const { uploadUrl, fileKey } = await response.json();

    // Upload to S3 with progress tracking
    const uploadResponse = await fetch(uploadUrl, {
      method: 'PUT',
      body: file,
      headers: {
        'Content-Type': file.type,
      },
    });

    if (!uploadResponse.ok) {
      throw new Error('Failed to upload file');
    }

    // Complete upload
    const completeResponse = await fetch('/api/uploads/complete', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        fileName: file.name,
        fileKey,
        fileType: file.type,
        fileSize: file.size,
        purpose,
      }),
    });

    if (!completeResponse.ok) {
      throw new Error('Failed to complete upload');
    }

    const { upload } = await completeResponse.json();
    onUploadComplete?.(upload);
    setProgress(100);
  };

  return (
    <div className="space-y-4">
      <input
        ref={fileInputRef}
        type="file"
        accept={accept.join(',')}
        multiple={multiple}
        onChange={(e) => e.target.files && handleFileSelect(e.target.files)}
        className="hidden"
      />

      <div
        onClick={() => fileInputRef.current?.click()}
        className="border-2 border-dashed border-gray-300 rounded-lg p-8 text-center cursor-pointer hover:border-gray-400 transition-colors"
      >
        {uploading ? (
          <div className="space-y-2">
            <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600 mx-auto" />
            <p>Uploading... {progress}%</p>
            {progress > 0 && (
              <div className="w-full bg-gray-200 rounded-full h-2">
                <div
                  className="bg-blue-600 h-2 rounded-full transition-all duration-300"
                  style={{ width: `${progress}%` }}
                />
              </div>
            )}
          </div>
        ) : (
          <div className="space-y-2">
            <CloudUploadIcon className="h-12 w-12 text-gray-400 mx-auto" />
            <div>
              <p className="text-lg font-medium">
                Click to upload {multiple ? 'files' : 'a file'}
              </p>
              <p className="text-sm text-gray-500">
                {accept.join(', ')} up to {Math.round(maxSize / 1024 / 1024)}MB
              </p>
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
```

### **File Gallery Component**
```typescript
// components/uploads/file-gallery.tsx
'use client';

interface FileGalleryProps {
  uploads: Upload[];
  onDelete?: (uploadId: string) => void;
  variant?: 'grid' | 'list';
}

export function FileGallery({ 
  uploads, 
  onDelete, 
  variant = 'grid' 
}: FileGalleryProps) {
  const [selectedFile, setSelectedFile] = useState<Upload | null>(null);

  const handleDelete = async (uploadId: string) => {
    if (!confirm('Are you sure you want to delete this file?')) return;

    try {
      await fetch(`/api/uploads/${uploadId}`, { method: 'DELETE' });
      onDelete?.(uploadId);
    } catch (error) {
      toast.error('Failed to delete file');
    }
  };

  if (variant === 'list') {
    return (
      <div className="space-y-2">
        {uploads.map((upload) => (
          <div
            key={upload.id}
            className="flex items-center justify-between p-3 border rounded-lg"
          >
            <div className="flex items-center space-x-3">
              {upload.fileType.startsWith('image/') ? (
                <img
                  src={upload.url}
                  alt={upload.fileName}
                  className="w-10 h-10 object-cover rounded"
                />
              ) : (
                <FileIcon className="w-10 h-10 text-gray-400" />
              )}
              <div>
                <p className="font-medium">{upload.fileName}</p>
                <p className="text-sm text-gray-500">
                  {formatFileSize(upload.fileSize)} • {format(upload.createdAt, 'MMM d, yyyy')}
                </p>
              </div>
            </div>
            <div className="flex space-x-2">
              <Button
                variant="outline"
                size="sm"
                onClick={() => window.open(upload.url, '_blank')}
              >
                View
              </Button>
              {onDelete && (
                <Button
                  variant="outline"
                  size="sm"
                  onClick={() => handleDelete(upload.id)}
                >
                  Delete
                </Button>
              )}
            </div>
          </div>
        ))}
      </div>
    );
  }

  return (
    <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
      {uploads.map((upload) => (
        <div
          key={upload.id}
          className="relative group border rounded-lg overflow-hidden"
        >
          {upload.fileType.startsWith('image/') ? (
            <img
              src={upload.url}
              alt={upload.fileName}
              className="w-full h-32 object-cover cursor-pointer"
              onClick={() => setSelectedFile(upload)}
            />
          ) : (
            <div className="w-full h-32 flex items-center justify-center bg-gray-100">
              <FileIcon className="w-8 h-8 text-gray-400" />
            </div>
          )}

          <div className="p-2">
            <p className="text-sm font-medium truncate">{upload.fileName}</p>
            <p className="text-xs text-gray-500">
              {formatFileSize(upload.fileSize)}
            </p>
          </div>

          {onDelete && (
            <button
              onClick={() => handleDelete(upload.id)}
              className="absolute top-2 right-2 bg-red-500 text-white p-1 rounded-full opacity-0 group-hover:opacity-100 transition-opacity"
            >
              <XIcon className="w-4 h-4" />
            </button>
          )}
        </div>
      ))}

      {/* Image viewer modal */}
      {selectedFile && selectedFile.fileType.startsWith('image/') && (
        <div
          className="fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center z-50"
          onClick={() => setSelectedFile(null)}
        >
          <div className="max-w-4xl max-h-4xl p-4">
            <img
              src={selectedFile.url}
              alt={selectedFile.fileName}
              className="max-w-full max-h-full object-contain"
            />
          </div>
        </div>
      )}
    </div>
  );
}

function formatFileSize(bytes: number): string {
  if (bytes === 0) return '0 Bytes';
  const k = 1024;
  const sizes = ['Bytes', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
}
```

---

## 🧪 **Upload Testing**

### **Upload Service Testing**
```typescript
// services/__tests__/upload.service.test.ts
describe('UploadService', () => {
  describe('generatePresignedUrl', () => {
    it('should generate presigned URL for valid file', async () => {
      const result = await UploadService.generatePresignedUrl({
        userId: 'user_123',
        fileName: 'test.jpg',
        fileType: 'image/jpeg',
        fileSize: 1024 * 1024, // 1MB
      });

      expect(result.uploadUrl).toContain('s3.amazonaws.com');
      expect(result.fileKey).toMatch(/uploads\/user_123\/\d+-[a-z0-9]+\.jpg/);
    });

    it('should reject invalid file type', async () => {
      await expect(
        UploadService.generatePresignedUrl({
          userId: 'user_123',
          fileName: 'test.exe',
          fileType: 'application/x-executable',
          fileSize: 1024,
        })
      ).rejects.toThrow('File type not allowed');
    });
  });
});
```

---

## 📋 **Upload Integration Checklist**

When implementing file uploads:
- [ ] Set up storage provider (S3/Cloudinary)
- [ ] Implement presigned URL generation
- [ ] Add file type and size validation
- [ ] Create upload completion tracking
- [ ] Add image processing if needed
- [ ] Implement file deletion
- [ ] Create upload UI components
- [ ] Add progress tracking
- [ ] Set up CDN for file delivery
- [ ] Write comprehensive tests
- [ ] Add proper error handling
- [ ] Implement file cleanup jobs

---

**This upload context provides complete file upload and storage patterns. Only load when file upload features are required.**