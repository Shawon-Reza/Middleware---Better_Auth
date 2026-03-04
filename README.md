# Middleware---Better_Auth

```jsx

import { Request, Response, NextFunction } from "express";
import { auth } from "../auth"; // your betterAuth instance

export const requireAuth = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  try {
    const session = await auth.api.getSession({
      headers: req.headers,
    });

    if (!session) {
      return res.status(401).json({
        message: "Unauthorized. Please login first.",
      });
    }

    // attach user to request
    (req as any).user = session.user;

    next();
  } catch (error) {
    return res.status(500).json({
      message: "Authentication failed",
    });
  }
};

```

# Email Verified Middleware
```jsx
export const requireVerifiedEmail = (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const user = (req as any).user;

  if (!user.emailVerified) {
    return res.status(403).json({
      message: "Please verify your email first.",
    });
  }

  next();
};
```

# Role Middleware (Multi Role)

```jsx
import { Request, Response, NextFunction } from "express";

export const requireRole =
  (...allowedRoles: string[]) =>
  (req: Request, res: Response, next: NextFunction) => {
    const user = (req as any).user;

    if (!user) {
      return res.status(401).json({ message: "Unauthorized" });
    }

    if (!allowedRoles.includes(user.role)) {
      return res
        .status(403)
        .json({ message: "Access denied. Insufficient role." });
    }

    next();
  };
```


## 🔐 Middleware Execution Order

Middlewares run in this order:

1. `requireAuth` → Check login  
2. `requireVerifiedEmail` → Check email verification  
3. `requireAdmin` → Check admin role  
4. Route handler runs  

If any middleware fails ❌ → request stops immediately.

