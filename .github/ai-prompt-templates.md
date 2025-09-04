# AI Prompt Templates for React Development

This document contains carefully crafted prompt templates for various React development scenarios. These prompts are designed to get the best results from AI assistants while ensuring security, performance, and best practices are followed.

## Component Generation Prompts

### Basic Component Creation

```
Create a React functional component with the following specifications:

**Component Details:**
- Component name: [COMPONENT_NAME]
- Purpose: [DESCRIPTION]
- Props: [LIST_PROPS_WITH_TYPES]
- Features: [LIST_FEATURES]

**Requirements:**
1. Use TypeScript with proper interfaces
2. Implement proper error handling and validation
3. Include accessibility features (ARIA labels, semantic HTML)
4. Add comprehensive JSDoc documentation
5. Include prop validation with TypeScript interfaces
6. Implement loading and error states if applicable
7. Follow React best practices (hooks, memoization)
8. Ensure responsive design considerations

**Security Requirements:**
- Sanitize all user inputs
- Validate all props and data
- Prevent XSS vulnerabilities
- Use secure coding practices

**Performance Considerations:**
- Use React.memo if component re-renders frequently
- Implement useMemo/useCallback where appropriate
- Consider code splitting if component is large
- Optimize for bundle size

Please provide:
1. The component code with TypeScript
2. Interface definitions
3. Usage examples
4. Performance notes
```

### Form Component Creation

```
Create a secure, accessible React form component with the following specifications:

**Form Details:**
- Form name: [FORM_NAME]
- Fields: [LIST_FIELDS_WITH_TYPES_AND_VALIDATION]
- Submit action: [DESCRIBE_SUBMISSION]
- Validation rules: [VALIDATION_REQUIREMENTS]

**Security Requirements:**
- Input sanitization and validation
- XSS prevention
- CSRF protection considerations
- Secure data handling

**Accessibility Requirements:**
- Proper form labels and associations
- Error announcements for screen readers
- Keyboard navigation support
- Focus management
- ARIA attributes

**Features to Include:**
- Real-time validation
- Loading states during submission
- Error handling and display
- Success/failure feedback
- Form reset functionality
- Dirty state tracking

**Technical Requirements:**
- TypeScript interfaces for all form data
- Custom form hook for logic
- Proper error boundaries
- Performance optimization
- Responsive design

Please provide:
1. Form component with TypeScript
2. Custom hook for form logic
3. Validation schema
4. Error handling implementation
5. Usage examples
```

### Data Display Component

```
Create a React component for displaying and managing data with the following specifications:

**Component Details:**
- Component name: [COMPONENT_NAME]
- Data type: [DATA_STRUCTURE]
- Display format: [TABLE/CARDS/LIST/GRID]
- Actions: [EDIT/DELETE/VIEW/FILTER/SORT]

**Data Management:**
- Filtering capabilities: [FILTER_TYPES]
- Sorting options: [SORT_FIELDS]
- Pagination: [PAGE_SIZE/INFINITE_SCROLL]
- Search functionality: [SEARCH_FIELDS]

**Performance Requirements:**
- Virtual scrolling for large datasets (>1000 items)
- Memoization for expensive operations
- Debounced search and filtering
- Optimized re-renders

**Security Considerations:**
- Input validation for filters/search
- XSS prevention in displayed content
- Access control for actions
- Safe HTML rendering

**Accessibility Features:**
- Keyboard navigation
- Screen reader support
- Focus management
- ARIA labels and descriptions
- High contrast support

Please provide:
1. Component implementation with TypeScript
2. Data interfaces and types
3. Performance optimizations
4. Accessibility features
5. Security measures
6. Integration examples
```

## Code Review Prompts

### Comprehensive Code Review

```
Perform a comprehensive code review of this React code:

**Code to Review:**
[PASTE_CODE_HERE]

**Review Areas:**
1. **Security Analysis:**
   - XSS vulnerabilities (dangerouslySetInnerHTML usage)
   - Input validation and sanitization
   - Authentication/authorization flaws
   - Sensitive data exposure
   - Injection vulnerabilities

2. **Performance Review:**
   - Unnecessary re-renders
   - Missing memoization opportunities
   - Expensive operations in render
   - Bundle size considerations
   - Memory leaks

3. **Best Practices Compliance:**
   - React patterns and conventions
   - TypeScript usage
   - Error handling
   - Component composition
   - State management

4. **Accessibility Audit:**
   - ARIA attributes
   - Semantic HTML
   - Keyboard navigation
   - Screen reader compatibility
   - Color contrast

5. **Code Quality:**
   - Readability and maintainability
   - Documentation
   - Test coverage
   - Error boundaries
   - Edge case handling

**Provide:**
1. Specific issues found with severity levels (Critical/High/Medium/Low)
2. Code examples showing fixes
3. Security recommendations
4. Performance optimizations
5. Accessibility improvements
6. Best practice suggestions
```

### Security-Focused Review

```
Perform a security audit of this React component/application:

**Code to Audit:**
[PASTE_CODE_HERE]

**Security Checklist:**
1. **Input Validation:**
   - All user inputs validated and sanitized
   - File upload restrictions and validation
   - URL parameter validation
   - Form data sanitization

2. **XSS Prevention:**
   - Safe use of dangerouslySetInnerHTML
   - User-generated content handling
   - External link safety
   - Script injection prevention

3. **Authentication & Authorization:**
   - Token storage security
   - Route protection implementation
   - Role-based access control
   - Session management

4. **Data Protection:**
   - Sensitive data not exposed in client
   - Environment variable usage
   - API communication security
   - Error message information leakage

5. **Content Security Policy:**
   - CSP header implementation
   - Unsafe inline scripts/styles
   - External resource loading

**Provide:**
1. Security vulnerabilities with CVSS scores
2. Specific remediation steps
3. Secure code examples
4. Security best practices recommendations
5. Compliance considerations (OWASP, etc.)
```

### Performance Review

```
Analyze this React code for performance optimization opportunities:

**Code to Analyze:**
[PASTE_CODE_HERE]

**Performance Analysis Areas:**
1. **Rendering Performance:**
   - Component re-render frequency
   - Expensive calculations in render
   - Missing memoization opportunities
   - List rendering optimization

2. **Memory Management:**
   - Memory leak detection
   - Object retention issues
   - Event listener cleanup
   - Large object handling

3. **Bundle Optimization:**
   - Code splitting opportunities
   - Tree shaking effectiveness
   - Large dependency analysis
   - Lazy loading potential

4. **Network Performance:**
   - API call optimization
   - Caching strategies
   - Resource loading
   - Image optimization

5. **User Experience:**
   - Loading state implementation
   - Progressive enhancement
   - Perceived performance
   - Core Web Vitals impact

**Provide:**
1. Performance issues with impact assessment
2. Optimization recommendations with code examples
3. Performance measurement strategies
4. Implementation priority (High/Medium/Low)
5. Expected performance improvements
```

## Architecture and Design Prompts

### Component Architecture Review

```
Review and suggest improvements for this React component architecture:

**Current Architecture:**
[DESCRIBE_CURRENT_STRUCTURE]

**Component Code:**
[PASTE_CODE_HERE]

**Architecture Review Areas:**
1. **Component Composition:**
   - Single Responsibility Principle adherence
   - Reusability and modularity
   - Prop interface design
   - Component coupling analysis

2. **State Management:**
   - Local vs. global state decisions
   - State normalization
   - Side effect management
   - Data flow patterns

3. **Performance Architecture:**
   - Memoization strategy
   - Code splitting approach
   - Bundle optimization
   - Lazy loading implementation

4. **Scalability Considerations:**
   - Component extension patterns
   - Maintainability factors
   - Testing architecture
   - Documentation structure

**Provide:**
1. Architectural improvement recommendations
2. Refactored component structure
3. Design pattern suggestions
4. Scalability improvements
5. Implementation roadmap
```

### Custom Hook Design

```
Design a custom React hook for the following use case:

**Hook Purpose:** [DESCRIBE_FUNCTIONALITY]
**Input Parameters:** [LIST_PARAMETERS]
**Return Values:** [LIST_RETURN_VALUES]
**Side Effects:** [DESCRIBE_SIDE_EFFECTS]

**Requirements:**
1. **Functionality:**
   - Core business logic implementation
   - Error handling and recovery
   - Loading state management
   - Caching and optimization

2. **API Design:**
   - Clean and intuitive interface
   - TypeScript support
   - Flexible configuration options
   - Backward compatibility

3. **Performance:**
   - Efficient re-render behavior
   - Memory leak prevention
   - Proper cleanup implementation
   - Optimization strategies

**Provide:**
1. Complete hook implementation with TypeScript
2. Usage examples and documentation
3. Performance considerations
4. API design rationale
```

## Debugging and Troubleshooting Prompts

### Error Diagnosis

```
Help diagnose and fix this React application error:

**Error Details:**
- Error message: [ERROR_MESSAGE]
- Stack trace: [STACK_TRACE]
- Browser/Environment: [BROWSER_INFO]
- Steps to reproduce: [REPRODUCTION_STEPS]

**Code Context:**
[PASTE_RELEVANT_CODE]

**Analysis Needed:**
1. **Root Cause Analysis:**
   - Identify the exact cause of the error
   - Analyze code flow leading to the error
   - Check for common React pitfalls

2. **Error Classification:**
   - Runtime error vs. build-time error
   - Client-side vs. server-side issue
   - Environment-specific problem

3. **Impact Assessment:**
   - User experience impact
   - Performance implications
   - Security considerations

**Solution Requirements:**
1. Immediate fix for the current error
2. Preventive measures for similar errors
3. Error boundary implementation if needed
4. Logging and monitoring improvements
5. Test cases to prevent regression

**Provide:**
1. Root cause explanation
2. Step-by-step fix implementation
3. Preventive code improvements
4. Error handling enhancements
5. Testing recommendations
```

### Performance Debugging

```
Help debug performance issues in this React application:

**Performance Problem:**
- Issue description: [DESCRIBE_PERFORMANCE_ISSUE]
- Affected components: [LIST_COMPONENTS]
- Performance metrics: [TIMING_DATA]
- User impact: [DESCRIBE_USER_IMPACT]

**Code to Analyze:**
[PASTE_CODE_HERE]

**Debugging Analysis:**
1. **Performance Profiling:**
   - Identify performance bottlenecks
   - Analyze component render cycles
   - Measure expensive operations
   - Check memory usage patterns

2. **Common Issues Investigation:**
   - Unnecessary re-renders
   - Expensive calculations in render
   - Memory leaks
   - Large bundle sizes

3. **Optimization Opportunities:**
   - Memoization potential
   - Code splitting benefits
   - Lazy loading opportunities
   - Asset optimization

**Solution Requirements:**
1. Specific performance optimizations
2. Before/after performance comparisons
3. Monitoring and measurement setup
4. Long-term performance strategy
5. User experience improvements

**Provide:**
1. Performance issue diagnosis
2. Optimization implementation plan
3. Code improvements with examples
4. Performance monitoring setup
5. Success metrics and validation
```

## General Prompts

### Code Quality Improvement

```
Improve the code quality of this React component following modern best practices:

**Current Code:**
[PASTE_CODE_HERE]

**Improvement Areas:**
1. **Code Structure:**
   - Component organization
   - Function decomposition
   - Import/export optimization
   - File structure improvements

2. **TypeScript Enhancement:**
   - Type safety improvements
   - Interface optimization
   - Generic usage
   - Type narrowing

3. **React Patterns:**
   - Hook usage optimization
   - Component composition
   - State management
   - Side effect handling

4. **Documentation:**
   - JSDoc comments
   - Type annotations
   - Usage examples
   - API documentation

**Quality Standards:**
- Follow ESLint and Prettier configurations
- Implement proper error boundaries
- Use consistent naming conventions
- Apply SOLID principles
- Ensure testability

**Provide:**
1. Refactored code with improvements
2. Explanation of changes made
3. Best practice recommendations
4. Documentation enhancements
```

### AI Prompt for Documentation

```
Add comprehensive JSDoc documentation to this code:

Requirements:
- Add JSDoc comments for all functions and components
- Include parameter descriptions and types
- Document return values
- Add usage examples for complex functions
- Include @throws annotations for error cases
- Add inline comments for complex logic
- Follow JSDoc best practices

Code to document:
[PASTE_CODE_HERE]
```

### AI Prompt for Route Structure Review

```
Review this route structure and navigation implementation:

Analyze the following aspects:
1. **Route Organization**: Module separation, naming consistency
2. **Type Safety**: Proper TypeScript usage for routes
3. **Parameter Handling**: Validation and error handling
4. **Protection**: Authentication and authorization patterns
5. **User Experience**: Breadcrumbs, fallbacks, mobile considerations
6. **Maintainability**: Scalability and code organization

Provide specific recommendations for:
- Route constant organization
- Dynamic route generation
- Error handling and fallbacks
- Performance optimizations
- Accessibility improvements

Route configuration:
[PASTE_ROUTE_CODE_HERE]
```

### AI Prompt Template for Best Practices Review
```
Review this React component for best practices compliance:

Analyze the following aspects:
1. **Code Organization**: File structure, naming conventions, component size
2. **Documentation**: JSDoc comments, prop descriptions, function documentation
3. **Performance**: Memoization usage, render optimization, unnecessary re-renders
4. **Type Safety**: TypeScript usage, prop validation, type definitions
5. **Accessibility**: ARIA attributes, semantic HTML, keyboard navigation
6. **State Management**: Appropriate state placement, context usage, side effects
7. **Security**: Input validation, XSS prevention, safe API calls

Provide specific recommendations with code examples for improvements.

Component code:
[PASTE_CODE_HERE]
```

### AI Prompt for Performance Optimization

```
Analyze this React component/application for performance optimization opportunities:

Performance areas to review:
1. **Rendering Performance**: Unnecessary re-renders, expensive calculations in render
2. **Memoization**: Missing useMemo, useCallback, React.memo opportunities
3. **Bundle Size**: Large imports, unused code, optimization opportunities
4. **Loading Performance**: Code splitting, lazy loading, asset optimization
5. **Memory Usage**: Memory leaks, large object retention, cleanup issues
6. **Network Performance**: API calls, caching strategies, request optimization
7. **User Experience**: Loading states, progressive enhancement, perceived performance
8. **Core Web Vitals**: LCP, FID, CLS optimization opportunities

Provide specific recommendations with:
- Code examples showing optimized versions
- Performance impact estimates
- Implementation priorities (high/medium/low impact)
- Measurement strategies to validate improvements

Code to analyze:
[PASTE_CODE_HERE]

Additional context:
- Expected data volumes: [SPECIFY]
- Target devices/browsers: [SPECIFY]
- Performance requirements: [SPECIFY]
```

### AI Prompt for Security Review

```
Perform a comprehensive security audit of this React component/application:

Security aspects to analyze:
1. **XSS Vulnerabilities**: Check for unsafe HTML rendering, missing input sanitization
2. **Input Validation**: Verify all user inputs are properly validated
3. **Authentication Flaws**: Review auth implementation and token handling
4. **Authorization Issues**: Check for proper access control
5. **API Security**: Analyze API calls for security headers and input sanitization
6. **Data Exposure**: Look for sensitive data leaks in client-side code
7. **CSRF Protection**: Verify CSRF tokens and protection mechanisms
8. **Content Security Policy**: Check CSP implementation
9. **File Upload Security**: Review file handling and validation
10. **Error Handling**: Ensure errors don't expose sensitive information

Provide specific security recommendations with secure code examples and explain the potential impact of each vulnerability found.

Code to review:
[PASTE_CODE_HERE]
```

### AI Prompt for Package Security Review

```
Review this package.json for security and maintenance best practices:

Analyze the following aspects:
1. **Security Vulnerabilities**: Check for known security issues
2. **Package Maintenance**: Evaluate update frequency and community support
3. **Bundle Size Impact**: Identify packages that significantly increase bundle size
4. **Dependency Conflicts**: Look for version conflicts or peer dependency issues
5. **Alternatives**: Suggest better-maintained or more secure alternatives
6. **Version Management**: Review version pinning strategy

Provide specific recommendations for:
- Packages to update or replace
- Security improvements
- Bundle size optimizations
- Maintenance workflow improvements

Package.json:
[PASTE_PACKAGE_JSON_HERE]

Current npm audit output:
[PASTE_AUDIT_OUTPUT_HERE]
```

### AI Prompt for Component Composition Review

```
Analyze this React component structure and suggest composition improvements:

Review areas:
1. **Reusability**: Can logic be extracted into reusable hooks or components?
2. **Composition patterns**: Are there opportunities for compound components, render props, or HOCs?
3. **Props interface**: Is the component API clean and focused?
4. **State management**: Is state appropriately located and managed?
5. **Coupling**: Are components too tightly coupled?
6. **Flexibility**: Can the component handle different use cases without modification?

Provide refactored code examples using appropriate composition patterns and explain the benefits of each suggested change.

Component code:
[PASTE_CODE_HERE]
```

---

## Usage Guidelines for AI Prompts

### Prompt Optimization Tips

1. **Be Specific:** Provide clear, detailed requirements
2. **Include Context:** Share relevant code, error messages, or constraints
3. **Set Expectations:** Define what you want in the response
4. **Prioritize Concerns:** Mention security, performance, and accessibility requirements
5. **Request Examples:** Ask for code examples and usage demonstrations

### Prompt Customization

- Replace placeholder text in brackets `[PLACEHOLDER]` with your specific requirements
- Adjust the scope and complexity based on your needs
- Add or remove sections based on your priorities
- Include your team's specific coding standards and preferences

### Quality Assurance

- Always review AI-generated code for security vulnerabilities
- Test generated components thoroughly
- Validate accessibility compliance
- Check performance implications
- Ensure code follows your team's standards

Remember: AI is a powerful assistant, but human review and validation are essential for production code.
