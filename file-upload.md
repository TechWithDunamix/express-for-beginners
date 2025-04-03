# File Uploads with Multer in Express.js

## File Uploads with Multer in Express.js

File uploads are a common requirement in web applications. Express.js doesn't handle file uploads natively, but Multer provides a straightforward middleware solution. This guide covers implementing file uploads with Multer in Express applications.

### Setting Up Multer

Multer is middleware for Express that makes handling `multipart/form-data` (file uploads) simple and efficient.

#### Installation

```javascript
// Terminal
npm install multer
```

#### Basic Configuration

```javascript
const express = require('express');
const multer = require('multer');
const path = require('path');

const app = express();

// Basic storage configuration
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, 'uploads/');
  },
  filename: function (req, file, cb) {
    cb(null, Date.now() + path.extname(file.originalname));
  }
});

// Initialize upload middleware
const upload = multer({ storage: storage });
```

### Simple File Upload Example

#### Single File Upload

```javascript
// Single file upload route
app.post('/upload', upload.single('file'), (req, res) => {
  if (!req.file) {
    return res.status(400).send('No file uploaded');
  }
  
  res.json({
    message: 'File uploaded successfully',
    file: {
      name: req.file.originalname,
      mimetype: req.file.mimetype,
      size: req.file.size,
      path: req.file.path
    }
  });
});
```

#### Multiple Files Upload

```javascript
// Multiple files upload route
app.post('/upload-multiple', upload.array('files', 5), (req, res) => {
  if (!req.files || req.files.length === 0) {
    return res.status(400).send('No files uploaded');
  }
  
  const fileDetails = req.files.map(file => ({
    name: file.originalname,
    mimetype: file.mimetype,
    size: file.size,
    path: file.path
  }));
  
  res.json({
    message: 'Files uploaded successfully',
    files: fileDetails
  });
});
```

#### Multiple Fields Upload

```javascript
// Upload multiple fields with different names
const multiUpload = upload.fields([
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 5 }
]);

app.post('/upload-profile', multiUpload, (req, res) => {
  if (!req.files) {
    return res.status(400).send('No files uploaded');
  }
  
  const response = {
    message: 'Files uploaded successfully',
    data: {}
  };
  
  if (req.files.avatar) {
    response.data.avatar = {
      name: req.files.avatar[0].originalname,
      path: req.files.avatar[0].path
    };
  }
  
  if (req.files.gallery) {
    response.data.gallery = req.files.gallery.map(file => ({
      name: file.originalname,
      path: file.path
    }));
  }
  
  res.json(response);
});
```

### Advanced Configuration

#### File Filtering

Control which files are accepted:

```javascript
// Filter middleware
const fileFilter = (req, file, cb) => {
  // Accept only image files
  if (file.mimetype.startsWith('image/')) {
    cb(null, true);
  } else {
    cb(new Error('Only image files are allowed'), false);
  }
};

// Use the filter
const upload = multer({
  storage: storage,
  fileFilter: fileFilter,
  limits: {
    fileSize: 1024 * 1024 * 5 // 5MB max file size
  }
});
```

#### Handling Errors

Add error handling for uploads:

```javascript
app.post('/upload', (req, res) => {
  upload.single('file')(req, res, function (err) {
    if (err instanceof multer.MulterError) {
      // A Multer error occurred
      if (err.code === 'LIMIT_FILE_SIZE') {
        return res.status(400).json({
          error: 'File too large, max size is 5MB'
        });
      }
      return res.status(400).json({ error: err.message });
    } else if (err) {
      // An unknown error occurred
      return res.status(500).json({ error: err.message });
    }
    
    // Everything went fine
    if (!req.file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }
    
    res.json({
      message: 'File uploaded successfully',
      file: {
        name: req.file.originalname,
        size: req.file.size,
        path: req.file.path
      }
    });
  });
});
```

### File Storage Strategies

#### Disk Storage

For storing files on the server's disk:

```javascript
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    // Create dynamic destination based on file type
    let uploadPath = 'uploads/';
    
    if (file.mimetype.startsWith('image/')) {
      uploadPath += 'images/';
    } else if (file.mimetype.startsWith('video/')) {
      uploadPath += 'videos/';
    } else {
      uploadPath += 'documents/';
    }
    
    // Ensure the directory exists
    const fs = require('fs');
    fs.mkdirSync(uploadPath, { recursive: true });
    
    cb(null, uploadPath);
  },
  filename: function (req, file, cb) {
    // Create a unique filename
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    const fileExt = path.extname(file.originalname);
    cb(null, file.fieldname + '-' + uniqueSuffix + fileExt);
  }
});
```

#### Memory Storage

For storing files in memory (useful for processing before saving):

```javascript
const storage = multer.memoryStorage();
const upload = multer({ storage: storage });

app.post('/upload-and-process', upload.single('image'), async (req, res) => {
  if (!req.file) {
    return res.status(400).send('No file uploaded');
  }
  
  try {
    // Process the file in memory before saving
    const sharp = require('sharp');
    const processedImageBuffer = await sharp(req.file.buffer)
      .resize(300, 300)
      .jpeg({ quality: 80 })
      .toBuffer();
    
    // Save the processed file
    const fs = require('fs').promises;
    const filename = Date.now() + '.jpg';
    await fs.writeFile('uploads/thumbnails/' + filename, processedImageBuffer);
    
    res.json({
      message: 'Image uploaded and processed',
      thumbnail: 'uploads/thumbnails/' + filename
    });
  } catch (err) {
    res.status(500).json({ error: 'Error processing image' });
  }
});
```

### Integrating with Cloud Storage

#### AWS S3 Integration

```javascript
// npm install multer-s3 aws-sdk

const aws = require('aws-sdk');
const multerS3 = require('multer-s3');

// Configure AWS
aws.config.update({
  accessKeyId: process.env.AWS_ACCESS_KEY_ID,
  secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
  region: process.env.AWS_REGION
});

const s3 = new aws.S3();

// Configure multer-s3
const uploadS3 = multer({
  storage: multerS3({
    s3: s3,
    bucket: process.env.S3_BUCKET,
    acl: 'public-read',
    contentType: multerS3.AUTO_CONTENT_TYPE,
    key: function (req, file, cb) {
      const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
      cb(null, 'uploads/' + uniqueSuffix + path.extname(file.originalname));
    }
  }),
  limits: { fileSize: 5 * 1024 * 1024 } // 5MB
});

// Use S3 upload middleware
app.post('/upload-to-s3', uploadS3.single('file'), (req, res) => {
  if (!req.file) {
    return res.status(400).send('No file uploaded');
  }
  
  res.json({
    message: 'File uploaded to S3 successfully',
    file: {
      name: req.file.originalname,
      location: req.file.location, // S3 URL
      key: req.file.key
    }
  });
});
```

### Security Considerations

#### File Size Validation

```javascript
const upload = multer({
  storage: storage,
  limits: {
    fileSize: 1024 * 1024 * 5, // 5MB
    files: 5 // Maximum 5 files per request
  }
});
```

#### Content Type Validation

```javascript
const fileFilter = (req, file, cb) => {
  // Allowed MIME types
  const allowedTypes = [
    'image/jpeg',
    'image/png',
    'image/gif',
    'application/pdf',
    'application/msword',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document'
  ];
  
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type'), false);
  }
};
```

#### File Extension Validation

```javascript
const fileFilter = (req, file, cb) => {
  // Allowed extensions
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif', '.pdf', '.doc', '.docx'];
  const fileExt = path.extname(file.originalname).toLowerCase();
  
  if (allowedExtensions.includes(fileExt)) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file extension'), false);
  }
};
```

#### Virus Scanning

For security-critical applications, scan files for viruses:

```javascript
// npm install clamscan

const ClamScan = require('clamscan');

const clamScan = new ClamScan({
  clamdscan: {
    socket: '/var/run/clamav/clamd.ctl',
    local_ip: '127.0.0.1',
    timeout: 60000
  },
  preference: 'clamdscan'
});

app.post('/upload-safe', upload.single('file'), async (req, res) => {
  if (!req.file) {
    return res.status(400).send('No file uploaded');
  }
  
  try {
    // Scan the uploaded file
    const { isInfected, file, viruses } = await clamScan.scan_file(req.file.path);
    
    if (isInfected) {
      // Delete the infected file
      fs.unlinkSync(req.file.path);
      return res.status(400).json({
        error: 'Virus detected',
        viruses: viruses
      });
    }
    
    // File is clean
    res.json({
      message: 'File uploaded and verified clean',
      file: {
        name: req.file.originalname,
        path: req.file.path
      }
    });
  } catch (err) {
    // Clean up the file if scanning fails
    fs.unlinkSync(req.file.path);
    res.status(500).json({ error: 'Error scanning file' });
  }
});
```

### File Upload Progress

Track upload progress for large files:

```javascript
// npm install multer socket.io

const http = require('http');
const socketIo = require('socket.io');

// Create http server
const server = http.createServer(app);
const io = socketIo(server);

// Track upload progress
app.post('/upload-with-progress', (req, res) => {
  // Create a socket channel for this upload
  const uploadId = Date.now().toString();
  
  // Track upload progress
  let uploadedBytes = 0;
  
  req.on('data', (chunk) => {
    uploadedBytes += chunk.length;
    io.emit(`upload-progress-${uploadId}`, {
      uploadedBytes,
      totalBytes: parseInt(req.headers['content-length'] || 0)
    });
  });
  
  // Process the upload
  upload.single('file')(req, res, function (err) {
    if (err) {
      return res.status(400).json({ error: err.message });
    }
    
    if (!req.file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }
    
    res.json({
      message: 'File uploaded successfully',
      file: {
        name: req.file.originalname,
        path: req.file.path
      }
    });
  });
});
```

### Creating a Full Upload Controller

```javascript
// controllers/uploadController.js
const multer = require('multer');
const path = require('path');

// Configure storage
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, 'uploads/');
  },
  filename: function (req, file, cb) {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
  }
});

// Configure file filter
const fileFilter = (req, file, cb) => {
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'application/pdf'];
  
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type'), false);
  }
};

// Create multer instance
const upload = multer({
  storage: storage,
  fileFilter: fileFilter,
  limits: {
    fileSize: 5 * 1024 * 1024 // 5MB
  }
});

// Upload controller methods
exports.uploadFile = (req, res) => {
  upload.single('file')(req, res, function (err) {
    if (err instanceof multer.MulterError) {
      return res.status(400).json({ error: err.message });
    } else if (err) {
      return res.status(500).json({ error: err.message });
    }
    
    if (!req.file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }
    
    res.json({
      message: 'File uploaded successfully',
      file: {
        name: req.file.originalname,
        mimetype: req.file.mimetype,
        size: req.file.size,
        path: req.file.path
      }
    });
  });
};

exports.uploadMultipleFiles = (req, res) => {
  upload.array('files', 5)(req, res, function (err) {
    if (err) {
      return res.status(400).json({ error: err.message });
    }
    
    if (!req.files || req.files.length === 0) {
      return res.status(400).json({ error: 'No files uploaded' });
    }
    
    const filesInfo = req.files.map(file => ({
      name: file.originalname,
      mimetype: file.mimetype,
      size: file.size,
      path: file.path
    }));
    
    res.json({
      message: 'Files uploaded successfully',
      files: filesInfo
    });
  });
};
```

### Serving Uploaded Files

Serve static files from your uploads directory:

```javascript
// Serve files from 'uploads' directory
app.use('/uploads', express.static(path.join(__dirname, 'uploads')));
```

For private files that require authentication:

```javascript
app.get('/files/:filename', authMiddleware, (req, res) => {
  const filePath = path.join(__dirname, 'uploads', req.params.filename);
  
  // Check if file exists
  fs.access(filePath, fs.constants.F_OK, (err) => {
    if (err) {
      return res.status(404).json({ error: 'File not found' });
    }
    
    // Serve the file
    res.sendFile(filePath);
  });
});
```

### Summary

* Multer provides middleware for handling file uploads in Express applications
* Configure storage strategies based on your needs: disk, memory, or cloud storage
* Filter uploads by file type, size, and extension for security
* Handle errors gracefully and provide clear feedback to users
* Create custom upload controllers for reusable upload functionality
* Consider scanning files for viruses in security-sensitive applications
* Track upload progress for large files to improve user experience
* Implement proper authentication for accessing uploaded files
