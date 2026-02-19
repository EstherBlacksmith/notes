The most solid approach is:

| Layer | Type to Use|  
|----------|----------|
| Domain Objects   | UUID and domain value type | 
| Repository Ports    | UUID and domain value type  |
| Repository Adapters   | Convert to String when calling JPA |
| JPA Entities    |  String (for portability)   | 
| JPA Repositories   |String (must match entity)  | 
