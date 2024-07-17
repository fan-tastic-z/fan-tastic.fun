---
title: "Rust_newtypes"
date: 2024-07-02T16:50:59+08:00
draft: true
---


<https://www.howtocodeit.com/articles/ultimate-guide-rust-newtypes> 阅读笔记

```rust
pub fn create_user(email: &str, password: &str) -> Result<User, CreateUserError> {
    validate_email(email)?;
    validate_password(password)?;

    let password_hash = hash_password(password)?;
    // Save user to database
    // Trigger welcome emails

    // ...
    Ok(User)
}
```

这种代码确实也是自己在日常对外的API 常用的方式，因为这里两个参数都是`&str`, 那对于调用者来说，如果不注意，其实比较容易传错。
同时因为可能存在的错误，我们就需要定义如此复杂的`CreateUserError`

```rust
#[derive(Error, Clone, Debug, PartialEq)]

pub enum CreateUserError {
    #[error("invalid email address: {0}")]
    InvalidEmail(String),
    #[error("invalid password: {reason}")]
    InvalidPassword {
        reason: String,
    },
    #[error("failed to hash password: {0}")]
    PasswordHashError(#[from] BcryptError),
    #[error("user with email address {email} already exists")]
    UserAlreadyExists {
        email: String,
    },
    // ...
}
```

随之而来的是复杂的单元测试，我们需要大量的测试用例来覆盖所有的结果：

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_create_user_invalid_email() {
        let email = "invalid-email";
        let password = "password";
        let result = create_user(email, password);
        let expected = Err(CreateUserError::InvalidEmail(email.to_string()));  
        assert_eq!(result, expected);
    }

    #[test]
    fn test_create_user_invalid_password() { unimplemented!() }

    #[test]
    fn test_create_user_password_hash_error() { unimplemented!() }

    #[test]
    fn test_create_user_user_already_exists() { unimplemented!() }

    #[test]
    fn test_create_user_success() { unimplemented!() }
}
```

并且这种复杂会导致如果函数内部对于参数的验证是部分验证成功，那么测试就会更加复杂，单元测试可能就会变成各种失败案例的排列组合。
