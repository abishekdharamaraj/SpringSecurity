## Security

Security is implemented with **Spring Security** using stateless **JWT (JSON Web Token)** authentication.

### How Authentication Works
1. The user registers or logs in through `/api/auth/register` or `/api/auth/login`.
2. On successful login, the server returns a signed JWT.
3. The client sends the token in the `Authorization` header on every protected request:
   ```
   Authorization: Bearer <your_token>
   ```
4. A JWT filter validates the token on each request and sets the authenticated user in the `SecurityContext`.
5. Spring Security checks the user's role before allowing access to the endpoint.

### Request Flow
```
Request → JwtAuthenticationFilter → SecurityContext → Controller
                  |
          Invalid or missing token → 401 Unauthorized
          Insufficient role        → 403 Forbidden
```

### Roles and Access Rules
| Role | Permissions |
|------|-------------|
| Public | Register, login, view products |
| `USER` | Manage own cart, place orders, view own order history |
| `ADMIN` | Manage products and categories, view all orders, update order status |

| Endpoint Pattern | Access |
|------------------|--------|
| `/api/auth/**` | Public |
| `GET /api/products/**` | Public |
| `POST, PUT, DELETE /api/products/**` | `ADMIN` |
| `/api/cart/**`, `/api/orders/**` | `USER` |
| `/api/admin/**` | `ADMIN` |
| `/swagger-ui/**`, `/v3/api-docs/**` | Public (disable in production) |

### Security Features
- Passwords hashed with **BCrypt** (never stored in plain text)
- Stateless sessions (`SessionCreationPolicy.STATELESS`)
- Token expiry configurable through `jwt.expiration`
- Role-based authorization using `hasRole()` and `@PreAuthorize`
- Custom handlers for `401 Unauthorized` and `403 Forbidden` responses
- CORS configuration for allowed frontend origins
- CSRF disabled, since the API is stateless and token-based
- Input validation to reduce injection risks (Bean Validation + JPA parameterized queries)

### Security Configuration
Key classes in the `security` and `config` packages:

| Class | Responsibility |
|-------|----------------|
| `SecurityConfig` | Defines the filter chain, route rules and password encoder |
| `JwtAuthenticationFilter` | Reads and validates the token on each request |
| `JwtUtil` / `JwtService` | Generates, parses and validates tokens |
| `CustomUserDetailsService` | Loads user details from the database |
| `JwtAuthEntryPoint` | Returns a JSON response for unauthorized access |

### Production Recommendations
- Store `jwt.secret` in an environment variable, not in `application.properties`
- Use a secret of at least 32 characters
- Serve the API over HTTPS only
- Restrict CORS to your frontend domain
- Disable Swagger UI in production
- Consider refresh tokens for longer sessions
