# Railway Deployment Plan for Google MCP Server

## Overview
This document outlines the deployment strategy for the Google MCP Server on Railway, a cloud platform that simplifies application deployment.

## Prerequisites
1. Railway account
2. Google Cloud Console access
3. Google OAuth 2.0 credentials
4. Git repository with the MCP server code

## Step 1: Prepare Google OAuth Credentials

### 1.1 Create Google OAuth Credentials
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Select your project or create a new one
3. Enable Google Docs API and Gmail API
4. Go to "Credentials" → "Create Credentials" → "OAuth 2.0 Client ID"
5. Select "Web application"
6. Add authorized redirect URI: `http://localhost:8080/`
7. Download the JSON file and save it as `credentials.json`

### 1.2 Generate Access Token
1. Run the authentication script locally:
   ```bash
   python auth.py
   ```
2. This will open a browser for OAuth authentication
3. After successful authentication, a `token.json` file will be created

## Step 2: Configure Railway Environment Variables

### 2.1 Required Environment Variables
Set these in your Railway project settings:

| Variable | Value | Description |
|----------|-------|-------------|
| `GOOGLE_CREDENTIALS_JSON` | *JSON content* | Your OAuth 2.0 client credentials |
| `GOOGLE_TOKEN_JSON` | *JSON content* | Your access token from auth.py |
| `AUTO_APPROVE` | `"true"` | Auto-approve actions in production |
| `PYTHON_VERSION` | `"3.11.9"` | Python runtime version |

### 2.2 How to Set Environment Variables
1. Go to your Railway project
2. Click on "Variables" tab
3. Add each variable with its value
4. For JSON variables, paste the entire content of the JSON files

## Step 3: Railway Configuration Files

### 3.1 railway.json
Create a `railway.json` file in your root directory:
```json
{
  "build": {
    "builder": "NIXPACKS"
  },
  "deploy": {
    "startCommand": "uvicorn server:app --host 0.0.0.0 --port $PORT",
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

### 3.2 nixpacks.toml 
Create a `nixpacks.toml` file for custom build configuration:
```toml
[phases.setup]
nixPkgs = ["python311"]

[phases.build]
cmds = ["pip install -r requirements.txt"]

[start]
cmd = "uvicorn server:app --host 0.0.0.0 --port $PORT"
```

## Step 4: Deployment Steps

### 4.1 Connect Repository to Railway
1. Sign in to [Railway](https://railway.app/)
2. Click "New Project"
3. Connect your GitHub repository
4. Select the repository containing your MCP server

### 4.2 Configure Deployment
1. Railway will automatically detect the Python application
2. Set the environment variables as outlined in Step 2
3. Configure the deployment settings:
   - **Build Command**: `pip install -r requirements.txt`
   - **Start Command**: `uvicorn server:app --host 0.0.0.0 --port $PORT`
   - **Port**: Railway will automatically assign a port

### 4.3 Deploy
1. Click "Deploy" to start the deployment process
2. Railway will build and deploy your application
3. Monitor the deployment logs for any issues

## Step 5: Post-Deployment Verification

### 5.1 Health Check
1. Once deployed, visit your Railway URL
2. Check the health endpoint: `https://your-app.railway.app/`
3. You should see: `{"message": "Google MCP Server is running 🚀"}`

### 5.2 Test MCP Tools
1. Check available tools: `https://your-app.railway.app/tools`
2. Test the endpoints with proper authentication

## Step 6: Monitoring and Maintenance

### 6.1 Logs
- Monitor Railway logs for any errors
- Check authentication and API call success rates

### 6.2 Token Refresh
- Google tokens expire periodically
- You may need to regenerate tokens and update `GOOGLE_TOKEN_JSON`
- Consider implementing automatic token refresh logic

### 6.3 Scaling
- Railway automatically scales based on demand
- Monitor performance and upgrade plans if needed

## Troubleshooting

### Common Issues

1. **Authentication Errors**
   - Verify `GOOGLE_CREDENTIALS_JSON` is correctly set
   - Check if `GOOGLE_TOKEN_JSON` is valid and not expired
   - Ensure both APIs are enabled in Google Cloud Console

2. **Build Failures**
   - Check `requirements.txt` for correct dependencies
   - Verify Python version compatibility
   - Review build logs for specific errors

3. **Runtime Errors**
   - Check environment variables are properly set
   - Verify the start command is correct
   - Review application logs

### Debug Commands
```bash
# Check environment variables
python -c "import os; print('GOOGLE_CREDENTIALS_JSON:', bool(os.environ.get('GOOGLE_CREDENTIALS_JSON')))"

# Test authentication locally
python -c "from auth import get_creds; print('Auth test:', get_creds().valid)"

# Verify server startup
uvicorn server:app --host 0.0.0.0 --port 8000
```

## Security Considerations

1. **Environment Variables**
   - Never commit credentials to git
   - Use Railway's encrypted environment variables
   - Rotate credentials periodically

2. **API Security**
   - The server auto-approves actions in production
   - Consider implementing additional authentication layers
   - Monitor API usage and access patterns

3. **Google OAuth**
   - Restrict OAuth scopes to minimum required
   - Set up proper redirect URIs
   - Monitor OAuth consent screen usage

## Cost Management

- Railway's free tier includes:
  - 500 hours of runtime per month
  - 100GB of egress bandwidth
  - Shared CPU and memory

- Monitor usage to avoid overages
- Consider upgrading to paid plans for production workloads

## Alternative Deployment Options

If Railway doesn't meet your needs, consider:

1. **Render** (already configured)
2. **Vercel** (with serverless functions)
3. **DigitalOcean App Platform**
4. **AWS Elastic Beanstalk**
5. **Google Cloud Run**

Each platform has different pricing and configuration requirements.
