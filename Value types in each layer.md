The most solid approach is:
<img width="893" height="131" alt="image" src="https://github.com/user-attachments/assets/beb64fc0-9b23-450f-b11d-16e3a48fb6e9" />

| Layer | Type to Use|  
|----------|----------|
| Domain Objects   | UUID and domain value type | 
| Repository Ports    | UUID and domain value type  |
| Repository Adapters   | Convert to String when calling JPA |
| JPA Entities    |  String (for portability)   | 
| JPA Repositories   |String (must match entity)  | 
