# Security Guidelines for React Applications

## Overview

Security in React applications requires a multi-layered approach covering client-side protection, secure API communication, authentication, and data validation. This guide provides comprehensive security practices for React development.

## Input Validation and Sanitization

### Client-Side Input Validation

```typescript
// ✅ Comprehensive input validation
interface IValidationRules {
  required?: boolean;
  minLength?: number;
  maxLength?: number;
  pattern?: RegExp;
  custom?: (value: any) => string | null;
}

// ✅ Simple input validation rules
const ValidationSchema = {
  email: {
    required: true,
    maxLength: 254,
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    custom: (v: string) => v.includes('..') ? 'No consecutive dots' : null
  },
  password: {
    required: true,
    minLength: 8,
    maxLength: 128,
    custom: (v: string) =>
      /(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(v)
        ? null
        : 'Must contain lowercase, uppercase, and number'
  },
  username: {
    required: true,
    minLength: 3,
    maxLength: 20,
    pattern: /^[a-zA-Z0-9_-]+$/,
    custom: (v: string) =>
      /^[_-]|[_-]$/.test(v)
        ? 'Cannot start or end with _ or -'
        : null
  }
};

const validateField = (value: any, rules: IValidationRules): string | null => {
  if (rules.required && !value?.trim()) return 'Required';
  if (rules.minLength && value.length < rules.minLength) return `Min ${rules.minLength}`;
  if (rules.maxLength && value.length > rules.maxLength) return `Max ${rules.maxLength}`;
  if (rules.pattern && !rules.pattern.test(value)) return 'Invalid format';
  if (rules.custom) return rules.custom(value);
  return null;
};
```

### Secure Form Components

Keep form inputs safe by sanitizing values and showing errors clearly.

```typescript
// ✅ Secure input component (simple version)
interface ISecureInputProps {
  name: string;
  type?: string;
  value: string;
  onChange: (name: string, value: string) => void;
  error?: string;
  maxLength?: number;
}

const SecureInput: React.FC<ISecureInputProps> = ({
  name,
  type = 'text',
  value,
  onChange,
  error,
  maxLength
}) => {
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  // Use your own sanitization function here
  const sanitized = sanitizeInput(e.target.value, maxLength);
  onChange(name, sanitized);
};

  return (
    <div>
      <input
        type={type}
        name={name}
        value={value}
        onChange={handleChange}
        maxLength={maxLength}
        aria-invalid={!!error}
        aria-describedby={error ? `${name}-error` : undefined}
      />
      {error && (
        <span id={`${name}-error`} role="alert">
          {error}
        </span>
      )}
    </div>
  );
};
```

**Best practices:**  
- Sanitize input to block scripts and null bytes  
- Limit input length  
- Show errors with accessible markup  

### File Upload Security

Validate file uploads for type, size, and extension before accepting.

```typescript
// ✅ Simple secure file upload
interface ISecureFileUploadProps {
  onFileSelect: (file: File) => void;
  acceptedTypes: string[];
  maxSizeBytes: number;
  onError: (error: string) => void;
}

const SecureFileUpload: React.FC<ISecureFileUploadProps> = ({
  onFileSelect,
  acceptedTypes,
  maxSizeBytes,
  onError
}) => {
  // You can create your own validation logic as needed
  const validateFile = (file: File): string | null => {
    // Example: check type and size
    if (file.size > maxSizeBytes) return 'File too large';
    if (!acceptedTypes.includes(file.type)) return 'Invalid file type';
    return null;
  };

  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;
    const error = validateFile(file);
    if (error) {
      onError(error);
      e.target.value = '';
      return;
    }
    onFileSelect(file);
  };

  return (
    <input
      type="file"
      onChange={handleFileChange}
      accept={acceptedTypes.join(',')}
    />
  );
};
```

> **Tip:** Adjust `validateFile` to match your security and business requirements.

## XSS Prevention

### Safe HTML Rendering

Never render user content as raw HTML without sanitization.

```typescript
// ❌ Dangerous: Direct HTML injection (avoid!)
const DangerousComponent = ({ userContent }: { userContent: string }) => (
  <div dangerouslySetInnerHTML={{ __html: userContent }} />
);

// ✅ Safe: Render as plain text
const SafeTextComponent = ({ userContent }: { userContent: string }) => (
  <div>{userContent}</div>
);

// ✅ Safe: Sanitized HTML with DOMPurify
import DOMPurify from 'dompurify';

const SafeHTMLComponent = ({ htmlContent }: { htmlContent: string }) => (
  <div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(htmlContent) }} />
);
```

**Instructions:**  
- Always sanitize user-generated HTML before rendering (use libraries like DOMPurify).
- Prefer plain text rendering when possible.
- Never use `dangerouslySetInnerHTML` with unsanitized content.

### URL Validation and Safe Links

Always validate URLs before rendering links, and use safe attributes for external links.

```typescript
// ✅ Safe external link component
interface ISafeLinkProps {
  href: string;
  children: React.ReactNode;
  className?: string;
  openInNewTab?: boolean;
}

const SafeLink: React.FC<ISafeLinkProps> = ({
  href,
  children,
  className,
  openInNewTab = true
}) => {
  // Block unsafe protocols and suspicious links
  if (/^(javascript:|data:|vbscript:)/i.test(href)) {
    return <span className={className}>{children}</span>;
  }

  const isInternal = href.startsWith('/') || href.startsWith('#');
  const props = {
    href,
    className,
    ...(openInNewTab && !isInternal && {
      target: '_blank',
      rel: 'noopener noreferrer'
    })
  };

  return isInternal ? <a {...props}>{children}</a> : <a {...props}>{children}</a>;
};
```

**Best practices:**  
- Block `javascript:`, `data:`, and other unsafe protocols  
- Use `rel="noopener noreferrer"` for external links  
- Prefer internal links for navigation  

## Authentication and Authorization

### Secure Token Management

**Best Practices:**

- **Store tokens securely:**  
  - Use httpOnly cookies for access and refresh tokens whenever possible (prevents XSS attacks).
  - Avoid storing sensitive tokens in `localStorage` or `sessionStorage`—these are accessible to JavaScript and vulnerable to XSS.

- **Token rotation:**  
  - Implement refresh tokens and rotate them regularly to reduce risk if a token is compromised.
  - Always validate tokens on the server before granting access.

- **Token revocation:**  
  - Provide a way to revoke tokens (e.g., on logout or password change).
  - Maintain a blacklist of revoked tokens on the server if needed.

- **Expiration and renewal:**  
  - Set short lifetimes for access tokens and require renewal via refresh tokens.
  - Prompt users to re-authenticate if both tokens expire.

- **Never expose secrets:**  
  - Do not embed secrets or sensitive keys in client-side code or environment variables prefixed with `REACT_APP_`.

- **Secure transmission:**  
  - Always send tokens over HTTPS.
  - Use the `SameSite` and `Secure` cookie flags for cookies.

- **Session management:**  
  - Clear tokens and sensitive data from memory and storage on logout.
  - Invalidate sessions on the server when users log out or change credentials.

**Summary:**  
Use httpOnly cookies for tokens, rotate and revoke tokens as needed, never expose secrets in client code, and always validate tokens server-side. This helps protect your app from XSS, CSRF, and token theft.

### Route Protection

Protect sensitive routes by requiring authentication, roles, or permissions.

```typescript
// ✅ Simple protected route example
const ProtectedRoute = ({ children, requiredRole }) => {
  const { user, isAuthenticated } = useAuth();
  if (!isAuthenticated) return <Navigate to="/login" />;
  if (requiredRole && !user.roles.includes(requiredRole)) return <Navigate to="/unauthorized" />;
  return <>{children}</>;
};

// Usage
<Route path="/admin" element={
  <ProtectedRoute requiredRole="admin">
    <AdminPanel />
  </ProtectedRoute>
} />
```

**Best Practices:**
- Require authentication for all protected routes.
- Check user roles and permissions before granting access.
- Redirect unauthenticated users to the login page.
- Redirect unauthorized users to an error or "unauthorized" page.
- Always validate access on the server as well.

**Summary:**  
Always enforce authentication and authorization for sensitive routes. Use both client-side checks for user experience and server-side checks for security.

## API Security

### Secure API Client Configuration

**Best Practices:**
- Always use HTTPS for all API requests.
- Set secure headers (e.g., `Content-Type`, `X-Requested-With`) for every request.
- Include authentication and CSRF tokens in requests as needed.
- Handle authentication and permission errors gracefully (e.g., redirect on 401/403).
- Never expose sensitive data in error messages or logs.
- Use `withCredentials: true` if cookies are needed for authentication.
- Validate and sanitize all data sent to and received from APIs.

```typescript
// ✅ Secure API client setup
const createSecureApiClient = () => {
  const client = axios.create({
    baseURL: process.env.REACT_APP_API_URL,
    timeout: 10000,
    withCredentials: true, // Include cookies
    headers: {
      'Content-Type': 'application/json',
      'X-Requested-With': 'XMLHttpRequest' // CSRF protection
    }
  });

  // Request interceptor for authentication
  client.interceptors.request.use(
    (config) => {
      const token = TokenStorage.getToken();
      if (token) {
        config.headers.Authorization = `Bearer ${token}`;
      }
      
      // Add CSRF token if available
      const csrfToken = getCsrfToken();
      if (csrfToken) {
        config.headers['X-CSRF-Token'] = csrfToken;
      }
      
      return config;
    },
    (error) => Promise.reject(error)
  );

  // Response interceptor for error handling
  client.interceptors.response.use(
    (response) => response,
    async (error) => {
      const { response } = error;
      
      if (response?.status === 401) {
        // Token expired, try to refresh
        try {
          await refreshToken();
          // Retry original request
          return client.request(error.config);
        } catch {
          // Refresh failed, redirect to login
          window.location.href = '/login';
        }
      }
      
      if (response?.status === 403) {
        // Forbidden - insufficient permissions
        window.location.href = '/unauthorized';
      }
      
      return Promise.reject(error);
    }
  );

  return client;
};

const api = createSecureApiClient();
```

**Summary:**  
Configure your API client to use secure headers, handle tokens, and manage errors. Always communicate with APIs over HTTPS and validate all data server-side. 

### Input Sanitization for API Calls

Sanitize all data before sending to APIs to prevent injection attacks and data corruption.

- Remove any scripts, suspicious patterns, or event handlers from string inputs.
- Limit the length of string fields to a reasonable maximum.
- Validate numbers to ensure they are finite and not NaN.
- Recursively sanitize arrays and objects to ensure nested data is clean.
- Always sanitize and validate API input on both client and server.
- Log errors without exposing sensitive data.

```typescript
// ✅ Secure API call wrapper
const secureApiCall = async <T>(
  method: 'GET' | 'POST' | 'PUT' | 'DELETE',
  endpoint: string,
  data?: Record<string, any>
): Promise<T> => {
  try {
    // Sanitize input data
    const sanitizedData = data ? sanitizeApiInput(data) : undefined;
    
    const response = await api.request({
      method,
      url: endpoint,
      data: sanitizedData
    });
    
    return response.data;
  } catch (error) {
    // Log error details (but not sensitive data)
    console.error(`API call failed: ${method} ${endpoint}`, {
      status: error.response?.status,
      message: error.message
    });
    
    throw error;
  }
};

// Usage
const createUser = async (userData: ICreateUserRequest): Promise<IUser> => {
  return secureApiCall<IUser>('POST', '/users', userData);
};
```

**Summary:**  
Always clean and validate all data before sending it to your APIs. This helps prevent security vulnerabilities and ensures your backend receives safe, expected input.

## Environment Variables Security

Environment variables are a key part of configuring React applications, but they come with important security considerations:

- **Public Exposure:**  
  All environment variables prefixed with `REACT_APP_` are exposed to the browser and can be read by anyone using developer tools. Never store secrets, API keys, passwords, or sensitive credentials in these variables.

- **Safe Usage:**  
  Use environment variables for non-sensitive configuration only, such as API URLs, app names, feature flags, and environment type.

- **Validation:**  
  Always validate your environment configuration before deploying to production. For example, ensure that your API URLs use HTTPS in production to prevent data interception.

- **Separation of Concerns:**  
  Keep environment-specific settings (development, staging, production) in separate `.env` files. Document what each variable does and why it’s needed.

- **Example:**
  ```typescript
  // ✅ Safe usage
  const apiUrl = process.env.REACT_APP_API_URL;

  // ❌ Unsafe: exposes secrets to client
  // const secretKey = process.env.REACT_APP_SECRET_KEY; // Accessible to anyone!
  ```

- **Configuration Validation Example:**
  ```typescript
  // Validate API URL for production
  if (
    process.env.REACT_APP_ENV === 'production' &&
    !process.env.REACT_APP_API_URL?.startsWith('https://')
  ) {
    throw new Error('API URL must use HTTPS in production');
  }
  ```

- **Best Practices:**
  - Never store secrets or sensitive data in client-side environment variables.
  - Use HTTPS for all API endpoints in production.
  - Document all environment variables and their intended use.
  - Validate configuration at startup and fail fast if insecure settings are detected.
  - Use server-side environment variables for secrets and sensitive data.

**Summary:**  
Environment variables in React are visible to the client. Use them only for safe, public configuration. Always validate and document your settings, and keep secrets on the server side.

## Security Headers and Best Practices

Security headers and utilities help protect your app from common web vulnerabilities.

- **Content Security Policy (CSP):**  
  Use a strong CSP header to block inline scripts and restrict resource loading. Generate a random nonce for each request to allow only trusted scripts.

- **Secure Random Generation:**  
  Use `crypto.getRandomValues` for generating secure random strings and nonces, especially for CSP and session tokens.

- **Time-Safe Comparison:**  
  Use time-safe string comparison to prevent timing attacks when comparing sensitive values (like tokens).

```typescript
// ✅ Security utilities (short version)
const SecurityUtils = {
  generateNonce: () =>
    btoa(String.fromCharCode(...crypto.getRandomValues(new Uint8Array(16)))),
  generateSecureRandom: (length: number) => {
    const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    return Array.from(crypto.getRandomValues(new Uint8Array(length)), b => chars[b % chars.length]).join('');
  },
  timeSafeCompare: (a: string, b: string) => {
    if (a.length !== b.length) return false;
    let result = 0;
    for (let i = 0; i < a.length; i++) result |= a.charCodeAt(i) ^ b.charCodeAt(i);
    return result === 0;
  }
};
```

**Best Practices:**  
- Set security headers like CSP, X-Frame-Options, X-Content-Type-Options, and Strict-Transport-Security.
- Use secure random values for tokens and nonces.
- Always use time-safe comparison for sensitive data.
- Review and update your security headers regularly.

**Summary:**  
Security headers and utilities are essential for defending against XSS, clickjacking, and other attacks. Use strong defaults and keep your configuration up to date.

## Security Checklist

### Pre-deployment Security Audit

- [ ] **Input Validation**
  - [ ] All user inputs are validated on client and server
  - [ ] File uploads are restricted and validated
  - [ ] URL parameters are validated
  - [ ] Form data is sanitized

- [ ] **XSS Prevention**
  - [ ] No use of `dangerouslySetInnerHTML` without sanitization
  - [ ] User-generated content is properly escaped
  - [ ] External links use `rel="noopener noreferrer"`
  - [ ] CSP headers are configured

- [ ] **Authentication & Authorization**
  - [ ] Tokens stored securely (httpOnly cookies)
  - [ ] Protected routes require authentication
  - [ ] Role-based access control implemented
  - [ ] Session timeout implemented

- [ ] **API Security**
  - [ ] All API calls use HTTPS
  - [ ] CSRF protection enabled
  - [ ] Rate limiting implemented
  - [ ] Input sanitization on API calls

- [ ] **Data Protection**
  - [ ] Sensitive data not exposed in client code
  - [ ] Environment variables properly configured
  - [ ] Error messages don't leak sensitive information
  - [ ] Logging doesn't include sensitive data

## Package Management and Security

### NPM Package Evaluation

**Before Installing Any Package:**
- **Check package reputation** - Look at download statistics, GitHub stars, and community adoption
- **Review security vulnerabilities** - Use `npm audit` and check security advisories
- **Evaluate maintenance status** - Check last update date, issue response time, and active maintainers
- **Assess bundle size impact** - Use tools like bundlephobia.com to check package size
- **Consider alternatives** - Compare similar packages for features, size, and maintenance

### Security Best Practices

```bash
# ✅ Always audit packages before and after installation
npm audit

# ✅ Fix high and critical vulnerabilities
npm audit fix

# ✅ Check for outdated packages
npm outdated

# ✅ Update packages safely
npm update

# ✅ Install exact versions for critical dependencies
npm install react@18.2.0 --save-exact
```

### Package Evaluation Checklist

**Before adding a new dependency:**
- [ ] Package has 1M+ weekly downloads (for popular utilities)
- [ ] Last updated within 6 months
- [ ] Has active maintenance (recent commits, issue responses)
- [ ] No critical security vulnerabilities
- [ ] Bundle size is reasonable for functionality
- [ ] TypeScript support available
- [ ] Good documentation and examples
- [ ] Compatible with current React version

### Package Security Commands

```typescript
// ✅ Check package bundle size before installing
const checkPackageSize = async (packageName: string) => {
  // Use bundlephobia.com API or check manually
  console.log(`Check bundle size: https://bundlephobia.com/package/${packageName}`);
};

// ✅ Security-focused package.json scripts
{
  "scripts": {
    "security:audit": "npm audit",
    "security:fix": "npm audit fix",
    "security:check": "npm audit --audit-level=moderate",
    "deps:check": "npm outdated",
    "deps:update": "npm update"
  }
}
```

### Dependency Management Guidelines

```typescript
// ✅ Prefer well-maintained, popular libraries
// Good choices for common needs:
const RECOMMENDED_PACKAGES = {
  dateHandling: 'date-fns', // vs moment.js (smaller, tree-shakable)
  httpClient: 'axios', // well-maintained, widely used
  formHandling: 'react-hook-form', // performant, good DX
  stateManagement: 'zustand', // simple, TypeScript-first
  ui: '@mui/material', // comprehensive, well-maintained
  icons: 'react-icons', // large collection, tree-shakable
  routing: 'react-router-dom', // standard for React
  testing: '@testing-library/react', // best practices focused
};

// ❌ Avoid packages with these red flags:
const AVOID_PACKAGES_WITH = [
  'No updates in 12+ months',
  'Critical security vulnerabilities',
  'Very small user base (<10k downloads/week)',
  'No TypeScript support',
  'Huge bundle size for simple functionality',
  'No documentation or examples',
  'Deprecated or abandoned status'
];
```

### Package Lock and Version Management

```json
// ✅ Use exact versions for critical dependencies
{
  "dependencies": {
    "react": "18.2.0",
    "react-dom": "18.2.0"
  },
  "devDependencies": {
    "typescript": "5.0.4",
    "@types/react": "18.2.6"
  }
}

// ✅ Set up automatic security updates
{
  "scripts": {
    "precommit": "npm audit --audit-level=high",
    "postinstall": "npm audit"
  }
}
```

### Security Monitoring

```typescript
// ✅ Set up automated security monitoring
// .github/workflows/security.yml
{
  name: "Security Audit",
  on: {
    push: { branches: ["main"] },
    pull_request: { branches: ["main"] },
    schedule: [{ cron: "0 0 * * 1" }] // Weekly
  },
  jobs: {
    security: {
      runs_on: "ubuntu-latest",
      steps: [
        { uses: "actions/checkout@v3" },
        { uses: "actions/setup-node@v3", with: { node_version: "18" } },
        { run: "npm ci" },
        { run: "npm audit --audit-level=moderate" }
      ]
    }
  }
}
```

### Alternative Package Evaluation

```typescript
// ✅ Compare packages before choosing
const PACKAGE_COMPARISON = {
  dateLibraries: {
    'date-fns': {
      bundleSize: '~13kb (tree-shakable)',
      typescript: 'Excellent',
      maintenance: 'Active',
      recommendation: 'Preferred for most cases'
    },
    'moment': {
      bundleSize: '~67kb (entire library)',
      typescript: 'Good',
      maintenance: 'Maintenance mode',
      recommendation: 'Avoid for new projects'
    },
    'dayjs': {
      bundleSize: '~2kb',
      typescript: 'Good',
      maintenance: 'Active',
      recommendation: 'Good for size-critical apps'
    }
  }
};
```

### Package Installation Workflow

```bash
# ✅ Recommended workflow for adding new packages

# 1. Research the package first
echo "1. Check package on npmjs.com, GitHub, and bundlephobia.com"

# 2. Check current security status
npm audit

# 3. Install the package
npm install <package-name>

# 4. Check for new vulnerabilities
npm audit

# 5. Update lock file and commit together
git add package.json package-lock.json
git commit -m "Add <package-name> dependency"

# 6. Test the application
npm test
npm run build
```

### Common Security Pitfalls

```typescript
// ❌ Don't ignore security warnings
// npm audit fix --force // Can break your app

// ✅ Address security issues properly
// 1. Check what the fix actually does
// 2. Test in development first
// 3. Update gradually, not all at once

// ❌ Don't use packages with obvious red flags
const SECURITY_RED_FLAGS = [
  'Packages with very few downloads',
  'Packages requesting unnecessary permissions',
  'Packages with suspicious recent changes',
  'Packages with no source code available',
  'Packages with multiple critical vulnerabilities'
];

// ✅ Use official packages when available
const PREFER_OFFICIAL = {
  '@types/react': 'Official TypeScript definitions',
  '@testing-library/react': 'Official testing utilities',
  'eslint-plugin-react': 'Official ESLint rules'
};
```

---

> **🔒 Security Note**: Always run `npm audit` before deploying to production. Set up automated security scanning in your CI/CD pipeline to catch vulnerabilities early. Keep dependencies updated, but test thoroughly before deploying updates to production.

## Summary

Security in React applications requires:

- **Defense in depth**: Multiple layers of security controls
- **Input validation**: Both client and server-side validation
- **Secure defaults**: Use secure configurations by default
- **Regular audits**: Continuous security assessment
- **Developer education**: Team awareness of security best practices

Remember: **Client-side security is not sufficient**. Always implement server-side validation and security controls as the primary defense mechanism.
