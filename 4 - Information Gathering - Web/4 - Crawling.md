# Crawling
## Extracting Valuable Information

Crawlers can extract a diverse array of data, each serving a specific purpose in the reconnaissance process:

- **Links (Internal or External)**
- **Comments:** Comments sections on blogs, forums, or other interactive pages can contain useful information. Users often inadvertently reveal sensitive details, internal processes, or hints of vulnerabilities in their comments.
- **Metadata:** Refers to "data about data". In context of web pages, it include information like titles, descriptions, keywords, author names, and dates. This metadata can provide valuable context about a page's content, purpose, and relevance to your reconnaissance goals.
- **Sensitive Files:** Web crawlers can be configured to actively search for files that might be exposed on a website. This include` backup files` (e.g. `.bak`, `.old`), `configuration files` (e.g., `web.config`, `settings.php`), `log files` (e.g., `error_log`, `access_log`), and other files containing `passwords`, `API Keys`. Examining extracted files (especially logs and backups) can reveal important data like `database credentials`, `encryption keys`, or even source code snippets.

```ad-note
Make sure to group the data and look for patterns, not just look at individual links. Doing this can help discovering a hidden chain of critical findings.

**e.g.** Seeing a comment mentioning a file server, then while looking at crawl rsults we see multiple links related to a **/files/** directory
```

# robots.txt

## What is robots.txt?

This file contains instructions in the form of "directives" that tell bots which parts of the website they can and cannot crawl.
### How robots.txt Works

The directives in robots.txt typically target specific user-agents, which are identifiers for different types of bots. For example, a directive might look like this:

```txt
User-agent: *
Disallow: /private/
```

# Well-Known URIs

The `.well-knows` typically is accessed via `/.well-known/` path on the web server, it centralizes a website's critical metadata, including config files and information related to its services, protocols, and security mechanisms. 

By establishing a consistent location for such data, `.well-known` simplifies the discovery and access process for various stakeholders, including web browsers, applications, and security tools. 

For instance, to access a website's security policy, a client would request `https://example.com/.well-known/security.txt`

The `Internet Assigned Numbers Authority` (`IANA`) maintains a [registry](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml) of `.well-known` URIs, each serving a specific purpose defined by various specifications and standards. Below is a table highlighting a few notable examples:

|URI Suffix|Description|Status|Reference|
|---|---|---|---|
|`security.txt`|Contains contact information for security researchers to report vulnerabilities.|Permanent|RFC 9116|
|`/.well-known/change-password`|Provides a standard URL for directing users to a password change page.|Provisional|[https://w3c.github.io/webappsec-change-password-url/#the-change-password-well-known-uri](https://w3c.github.io/webappsec-change-password-url/#the-change-password-well-known-uri)|
|`openid-configuration`|Defines configuration details for OpenID Connect, an identity layer on top of the OAuth 2.0 protocol.|Permanent|[http://openid.net/specs/openid-connect-discovery-1_0.html](http://openid.net/specs/openid-connect-discovery-1_0.html)|
|`assetlinks.json`|Used for verifying ownership of digital assets (e.g., apps) associated with a domain.|Permanent|[https://github.com/google/digitalassetlinks/blob/master/well-known/specification.md](https://github.com/google/digitalassetlinks/blob/master/well-known/specification.md)|
|`mta-sts.txt`|Specifies the policy for SMTP MTA Strict Transport Security (MTA-STS) to enhance email security.|Permanent|RFC 8461|

## Web Recon and .well-known

One particularly useful URI is `openid-configuration`.  When a client application wants to use OpenID Connect for authentication, it can retrieve the OpenID Connect Provider's configuration by accessing the `https://example.com/.well-known/openid-configuration` endpoint. This endpoint returns a JSON document containing metadata about the provider's endpoints, supported authentication methods, token issuance, and more:

```json
{ "issuer": "https://example.com", "authorization_endpoint": "https://example.com/oauth2/authorize", "token_endpoint": "https://example.com/oauth2/token", "userinfo_endpoint": "https://example.com/oauth2/userinfo", "jwks_uri": "https://example.com/oauth2/jwks", "response_types_supported": ["code", "token", "id_token"], "subject_types_supported": ["public"], "id_token_signing_alg_values_supported": ["RS256"], "scopes_supported": ["openid", "profile", "email"] }
```

The information obtained from the `openid-configuration` endpoint provides multiple exploration opportunities:

1. **Endpoint Discovery:** 
	- **Authorization Endpoint:** Identify URL for user authentication requests.
	- **Token Endpoint:** Finding the URL where tokens are issues.
	- **Userinfo Endpoint:** Locating the endpoint that provides user information
2. **JWKS URI**
3. **Supported Scopes and Responses Types**
4. **Algorithm Details**

