### Where Each Type of Validation Goes

| Validation Type   | Layer                     | Example                                   |
|-------------------|---------------------------|-------------------------------------------|
| Null/Empty        | DTO + Domain Entity       | `@NotNull` + constructor check            |
| Format            | DTO (regex) + Domain Entity | `@Email` + custom validation              |
| Business Rules    | Domain Service/Entity     | "Player name must be unique"              |
| Cross-Aggregate   | Domain Service            | "Player has enough balance"               |

## Key Principles:
- Defense in Depth: Validate at multiple layers
- Fail Fast: Validate as early as possible (DTO level)
- Domain Logic: Keep business rules in the domain layer
- Single Responsibility: Don't mix validation with business logic
