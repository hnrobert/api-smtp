# High-Performance SMTP API

A containerized FastAPI-based email sending service with attachment support, built for production use with Docker, MinIO object storage, and GitHub Actions CI/CD.

## ✨ Features

- **Email Delivery**: Send emails via SMTP with support for both plain text and HTML content
- **Attachment Support**: Upload and send files as email attachments (up to 2 files, 2MB each)
- **Object Storage**: MinIO integration for reliable attachment storage
- **Flexible Authentication**: Optional API key authentication (can be disabled for development)
- **Dual Email Configuration**: Separate SMTP authentication and display email addresses
- **Container Ready**: Full Docker Compose setup with Nginx, MailDev, and MinIO
- **CI/CD Pipeline**: Automated Docker image builds with GitHub Actions
- **Configuration**: JSONC format with comments for easy configuration management

## 🏗️ Project Structure

```text
api-smtp/
├── .github/
│   └── workflows/
│       └── docker-image.yml      # GitHub Actions CI/CD pipeline
├── src/
│   ├── app/
│   │   ├── main.py               # FastAPI application
│   │   ├── requirements.txt      # Python dependencies
│   │   └── smtp_config.jsonc     # Configuration file (JSONC with comments)
│   ├── docker/
│   │   └── Dockerfile           # Application container definition
│   └── nginx/
│       ├── nginx.conf           # Main Nginx configuration
│       └── conf.d/
│           └── default.conf     # Virtual host configuration
├── docker-compose.yml           # Multi-service container orchestration
├── .dockerignore               # Docker build context exclusions
└── readme.md                   # This file
```

## ⚙️ Configuration

Configuration is stored in `src/app/smtp_config.jsonc` (JSONC format with comments):

```jsonc
{
  // API Configuration
  "api_key": "", // Leave empty to disable authentication
  "api_name": "High-Performance SMTP API",
  "api_description": "SMTP API mail dispatch with support for attachments.",

  // SMTP Server Settings
  "smtp_server": "maildev", // SMTP server hostname
  "smtp_port": 1025, // SMTP port
  "use_ssl": false, // Use SSL/TLS encryption
  "use_password": false, // SMTP authentication required
  "use_tls": false, // Use STARTTLS

  // Email Limits
  "max_len_recipient_email": 64, // Max recipient email length
  "max_len_subject": 255, // Max subject length
  "max_len_body": 50000, // Max email body length

  // Sender Configuration
  "sender_email": "your_email@example.com", // SMTP auth email (actual account)
  "sender_email_display": "", // From header email (leave empty to use sender_email)
  "sender_domain": "devel.local.email",
  "sender_password": "your_password",

  // MinIO Object Storage Settings
  "minio_server": "minio:9000", // MinIO server endpoint
  "minio_access_key": "minioadmin", // MinIO access key
  "minio_secret_key": "minioadmin", // MinIO secret key
  "minio_secure": false // Use HTTPS for MinIO
}
```

### 🔐 Authentication Options

- **Enabled**: Set `api_key` to a secure value (e.g., UUID)
- **Disabled**: Set `api_key` to empty string `""` for development/testing

### 📧 Email Address Configuration

- **`sender_email`**: The actual SMTP account used for authentication
- **`sender_email_display`**: The email address shown to recipients (optional)

Example scenarios:

```jsonc
// Scenario 1: Use same email for both auth and display
"sender_email": "noreply@company.com",
"sender_email_display": "",

// Scenario 2: Different auth and display emails
"sender_email": "smtp-service@company.com",       // Technical account
"sender_email_display": "support@company.com",    // User-friendly display
```

Endpoint: `/v1/mail/send`
Method: `POST`
Content-Type: `application/json`

## 🚀 Quick Start

### Prerequisites

- Docker and Docker Compose
- Git

### Installation

1. **Clone the repository**:

   ```bash
   git clone https://github.com/hnrobert/api-smtp.git
   cd api-smtp
   ```

2. **Configure the application**:

   ```bash
   # Edit the configuration file
   nano src/app/smtp_config.jsonc
   ```

3. **Start all services**:

   ```bash
   docker-compose up -d
   ```

4. **Verify services are running**:

   ```bash
   docker-compose ps
   ```

### Service Access

- **API Documentation**: <http://localhost/docs> (Swagger UI)
- **Alternative Docs**: <http://localhost/redoc> (ReDoc)
- **MailDev Web Interface**: <http://localhost:1080> (Email testing)
- **MinIO Console**: <http://localhost:9001> (Object storage, login: minioadmin/minioadmin)

## 📡 API Usage

### 1. Send Email with Attachments

**Endpoint**: `POST /v1/mail/send-with-attachments`  
**Content-Type**: `multipart/form-data`

```bash
curl -X POST "http://localhost/v1/mail/send-with-attachments" \
  -H "X-API-Key: your_api_key" \
  -F "recipient_email=test@example.com" \
  -F "subject=Test with Attachment" \
  -F "body=<h1>Hello World!</h1>" \
  -F "body_type=html" \
  -F "debug=false" \
  -F "attachments=@/path/to/file.pdf"
```

### 2. Send Email (JSON)

**Endpoint**: `POST /v1/mail/send`  
**Content-Type**: `application/json`

```bash
curl -X POST "http://localhost/v1/mail/send" \
  -H "X-API-Key: your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "recipient_email": "test@example.com",
    "subject": "Test Email",
    "body": "Hello from API!",
    "body_type": "plain",
    "debug": false
  }'
```

### 3. No Authentication Example

If `api_key` is set to `""` in config:

```bash
curl -X POST "http://localhost/v1/mail/send" \
  -H "Content-Type: application/json" \
  -d '{
    "recipient_email": "test@example.com",
    "subject": "Test Email",
    "body": "No auth required!"
  }'
```

## 🏗️ Architecture

### Service Stack

- **Nginx**: Reverse proxy and load balancer
- **FastAPI App**: Main email sending application
- **MailDev**: SMTP server for development/testing
- **MinIO**: S3-compatible object storage for attachments
- **MinIO Client**: Automatic bucket setup

### Data Flow

1. **Request** → Nginx → FastAPI App
2. **Attachments** → MinIO Storage
3. **Email Assembly** → SMTP Server (MailDev)
4. **Logs** → Local file system (`./data/`)

## 🔧 Development

### Local Development

1. **Install dependencies**:

   ```bash
   pip install -r src/app/requirements.txt
   ```

2. **Run the app locally**:

   ```bash
   cd src/app
   python main.py
   ```

3. **Access at**: <http://localhost:8000>

### Testing

```bash
# Run with Docker Compose for full stack testing
docker-compose up --build

# Check logs
docker-compose logs api-smtp

# Test endpoints
curl -X GET "http://localhost/docs"
```

## 🚀 Production Deployment

### GitHub Actions CI/CD

This project includes automated CI/CD that builds and pushes Docker images to GitHub Container Registry (GHCR).

**Triggers**:

- Push to `main` branch → `latest` tag
- Push to `develop` branch → `develop` tag

**Only builds when these files change**:

- `.github/workflows/docker-image.yml`
- `src/docker/Dockerfile`
- `src/app/requirements.txt`
- `src/app/**/*.py`

### Using Pre-built Images

```yaml
# docker-compose.yml
services:
  api-smtp:
    image: ghcr.io/hnrobert/api-smtp:latest
    # ... rest of configuration
```

### Production Configuration

1. **Security**:

   ```jsonc
   {
     "api_key": "secure-random-uuid-here",
     "minio_access_key": "production-access-key",
     "minio_secret_key": "production-secret-key",
     "minio_secure": true
   }
   ```

2. **SMTP Settings**:

   ```jsonc
   {
     "smtp_server": "your-smtp-server.com",
     "smtp_port": 587,
     "use_tls": true,
     "use_password": true,
     "sender_email": "noreply@yourcompany.com",
     "sender_password": "your-smtp-password"
   }
   ```

## 🐛 Troubleshooting

### Common Issues

1. **"Could not validate credentials"**

   - Check if `api_key` is correctly set in config
   - Verify `X-API-Key` header in requests

2. **MinIO connection errors**

   - Ensure MinIO container is running: `docker-compose ps`
   - Check MinIO credentials in config

3. **SMTP errors**

   - Verify SMTP server settings
   - Check MailDev is running for development

4. **File upload failures**
   - Ensure file size < 2MB
   - Maximum 2 attachments per email

### Logs and Debugging

```bash
# View application logs
docker-compose logs api-smtp

# View all service logs
docker-compose logs

# Check specific container
docker logs api-smtp

# Debug mode in requests
# Add "debug": true to enable email debugging
```

## 📊 Monitoring

### Email Logs

Logs are stored in `./data/` directory:

```text
data/
├── 2025-09-22/
│   ├── success/
│   │   └── email-uuid.json
│   ├── failure/
│   │   └── email-uuid.json
│   └── debug/
│       └── email-uuid_email.txt
```

### Health Checks

```bash
# Check API health
curl http://localhost/docs

# Check MinIO
curl http://localhost:9001

# Check MailDev
curl http://localhost:1080
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature-name`
3. Make your changes
4. Test with: `docker-compose up --build`
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👨‍💻 Author

- **Robert He** - [hnrobert](https://github.com/hnrobert)

---

**⭐ Star this repo if you find it useful!**
