The following table shows which functions may execute files and which only read file content:

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |        ✅         |      ✅      |       ✅        |
| `require()`/`require_once()` |        ✅         |      ✅      |       ❌        |
| `file_get_contents()`        |        ✅         |      ❌      |       ✅        |
| `fopen()`/`file()`           |        ✅         |      ❌      |       ❌        |
| **NodeJS**                   |                  |             |                |
| `fs.readFile()`              |        ✅         |      ❌      |       ❌        |
| `fs.sendFile()`              |        ✅         |      ❌      |       ❌        |
| `res.render()`               |        ✅         |      ✅      |       ❌        |
| **Java**                     |                  |             |                |
| `include`                    |        ✅         |      ❌      |       ❌        |
| `import`                     |        ✅         |      ✅      |       ✅        |
| **.NET**                     |                  |             |                |
| `@Html.Partial()`            |        ✅         |      ❌      |       ❌        |
| `@Html.RemotePartial()`      |        ✅         |      ❌      |       ✅        |
| `Response.WriteFile()`       |        ✅         |      ❌      |       ❌        |
| `include`                    |        ✅         |      ✅      |       ✅        |
