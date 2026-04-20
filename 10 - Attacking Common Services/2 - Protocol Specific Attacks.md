# The Concept of Attacks

![[Pasted image 20260419232044.png]]

## Source

There are many different ways to pass information to a process

|**Information Source**|**Description**|
|---|---|
|`Code`|This means that the already executed program code results are used as a source of information. These can come from different functions of a program.|
|`Libraries`|A library is a collection of program resources, including configuration data, documentation, help data, message templates, prebuilt code and subroutines, classes, values, or type specifications.|
|`Config`|Configurations are usually static or prescribed values that determine how the process processes information.|
|`APIs`|The application programming interface (API) is mainly used as the interface of programs for retrieving or providing information.|
|`User Input`|If a program has a function that allows the user to enter specific values used to process the information accordingly, this is the manual entry of information by a person.|

## Processes

| **Process Components** | **Description**                                                                                                                                                            |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `PID`                  | The Process-ID (PID) identifies the process being started or is already running. Running processes have already assigned privileges, and new ones are started accordingly. |
| `Input`                | This refers to the input of information that could be assigned by a user or as a result of a programmed function.                                                          |
| `Data processing`      | The hard-coded functions of a program dictate how the information received is processed.                                                                                   |
| `Variables`            | The variables are used as placeholders for information that different functions can further process during the task.                                                       |
| `Logging`              | During logging, certain events are documented and, in most cases, stored in a register or a file. This means that certain information remains in the system.               |
## Privileges

| **Privileges** | **Description**                                                                                                                                                                                           |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `System`       | These privileges are the highest privileges that can be obtained, which allow any system modification. In Windows, this type of privilege is called `SYSTEM`, and in Linux, it is called `root`.          |
| `User`         | User privileges are permissions that have been assigned to a specific user. For security reasons, separate users are often set up for particular services during the installation of Linux distributions. |
| `Groups`       | Groups are a categorization of at least one user who has certain permissions to perform specific actions.                                                                                                 |
| `Policies`     | Policies determine the execution of application-specific commands, which can also apply to individual or grouped users and their actions.                                                                 |
| `Rules`        | Rules are the permissions to perform actions handled from within the applications themselves.                                                                                                             |
## Destination

|**Destination**|**Description**|
|---|---|
|`Local`|The local area is the system's environment in which the process occurred. Therefore, the results and outcomes of a task are either processed further by a process that includes changes to data sets or storage of the data.|
|`Network`|The network area is mainly a matter of forwarding the results of a process to a remote interface. This can be an IP address and its services or even entire networks. The results of such processes can also influence the route under certain circumstances.|
# Service Misconfigurations

## Authentication

```ad-tldr
After grabbing the service banner, always check for default credentials or weak used credentials (something like `admin:Password`) or Anonymous login.

---

Check for **Misconfigured Access Rights**, a user thats role is to upload files could have the permession to read everything on the file server which can expose critical configuration files, credentials, and PII
```
## Unnecessary Defaults

```ad-tldr
Sometimes administrators keep the default configs/settings for the software after installation, this can open doors for vulnerabilities targeting misconfigurations in applications.
```

# Finding Sensitive Information

Sensitive information may include, but is not limited to:

- Usernames.
- Email Addresses.
- Passwords.
- DNS records.
- IP Addresses.
- Source code.
- Configuration files.
- PII.

