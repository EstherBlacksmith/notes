## 📚 Common Validation Annotations

| Annotation        | Purpose                                               | Example                                                  |
|-------------------|-------------------------------------------------------|----------------------------------------------------------|
| `@NotNull`        | Field cannot be null                                  | `@NotNull String name`                                   |
| `@NotBlank`       | String cannot be blank (null, empty, or whitespace)   | `@NotBlank String name`                                  |
| `@Size`           | String or collection size constraints                 | `@Size(min = 2, max = 50) String name`                   |
| `@Min` / `@Max`   | Numeric minimum / maximum values                      | `@Min(1) @Max(100) int score`                            |
| `@Pattern`        | Regular expression pattern matching                   | `@Pattern(regexp = "^[a-zA-Z]+$") String name`           |
| `@Email`          | Valid email format                                    | `@Email String email`                                    |
