
```ad-info
Refer back to **Password Attacks** module and read parts related to `Username-Anarchy` and CUPP tools
```

# CUPP

**Example:**

| Field               | Details                      |
| ------------------- | ---------------------------- |
| Name                | Jane Smith                   |
| Nickname            | Janey                        |
| Birthdate           | December 11, 1990            |
| Relationship Status | In a relationship with Jim   |
| Partner's Name      | Jim (Nickname: Jimbo)        |
| Partner's Birthdate | December 12, 1990            |
| Pet                 | Spot                         |
| Company             | AHI                          |
| Interests           | Hackers, Pizza, Golf, Horses |
| Favorite Colors     | Blue                         |

```ad-tip
**Let's say the password policy in the domain was the following:**
- Minimum Length: 6 characters
- Must Include:
    - At least one uppercase letter
    - At least one lowercase letter
    - At least one number
    - At least two special characters (from the set `!@#$%^&*`)
      
We can filter the password list to be compatible with the password policy:

`grep -E '^.{6,}$' jane.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' | grep -E '([!@#$%^&*].*){2,}' > jane-filtered.txt`
```

