# GitHub REST API: Pull Requests

API reference documentation for managing pull requests using the GitHub REST API.

---

## Overview

The Pull Requests API allows you to list, create, update, and merge pull requests in a GitHub repository. Pull requests let you propose changes to a repository and request that a collaborator review and merge your contribution.

This documentation covers the most commonly used Pull Requests endpoints. For the full API reference, see the [official GitHub REST API documentation](https://docs.github.com/en/rest/pulls).

---

## Authentication

All requests to the GitHub REST API require authentication for full access. Unauthenticated requests are subject to stricter rate limits and cannot access private repositories.

### Personal Access Token (recommended)

Include a personal access token in the `Authorization` header:

```
Authorization: Bearer YOUR_TOKEN
```

Tokens can be generated at [github.com/settings/tokens](https://github.com/settings/tokens). For pull request operations, the token requires the `repo` scope.

### Example

```bash
curl -H "Authorization: Bearer ghp_abc123def456" \
  https://api.github.com/repos/octocat/hello-world/pulls
```

---

## Base URL

```
https://api.github.com
```

All endpoints in this document are relative to this base URL.

---

## Common Headers

Include these headers with every request:

| Header | Value | Purpose |
|--------|-------|---------|
| `Authorization` | `Bearer YOUR_TOKEN` | Authentication |
| `Accept` | `application/vnd.github+json` | Specifies the response format |
| `X-GitHub-Api-Version` | `2022-11-28` | Locks the API version to avoid breaking changes |

---

## Endpoints

### List Pull Requests

Retrieve all pull requests for a repository, with optional filters for state, sort order, and branch.

**Method:** `GET`

**Path:** `/repos/{owner}/{repo}/pulls`

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `owner` | string | Yes | The account owner of the repository (username or organization) |
| `repo` | string | Yes | The name of the repository |

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `state` | string | `open` | Filter by state. Options: `open`, `closed`, `all` |
| `sort` | string | `created` | What to sort results by. Options: `created`, `updated`, `popularity`, `long-running` |
| `direction` | string | `desc` | Sort direction. Options: `asc`, `desc` |
| `head` | string | none | Filter by the branch that contains the changes (format: `user:branch-name`) |
| `base` | string | none | Filter by the branch that the pull request targets |
| `per_page` | integer | 30 | Number of results per page (maximum: 100) |
| `page` | integer | 1 | Page number for paginated results |

#### Example Request

```bash
curl -H "Authorization: Bearer ghp_abc123def456" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/Uniswap/v4-core/pulls?state=open&per_page=5"
```

#### Example Response

```json
[
  {
    "id": 1234567890,
    "number": 1031,
    "state": "open",
    "title": "docs: expand SECURITY.md with disclosure policy and audit references",
    "user": {
      "login": "IAM-CW",
      "id": 12345678
    },
    "body": "This PR expands the existing SECURITY.md...",
    "created_at": "2026-03-23T10:30:00Z",
    "updated_at": "2026-03-23T10:30:00Z",
    "html_url": "https://github.com/Uniswap/v4-core/pull/1031",
    "head": {
      "ref": "docs/security-update",
      "sha": "a1b2c3d4e5f6"
    },
    "base": {
      "ref": "main",
      "sha": "f6e5d4c3b2a1"
    },
    "draft": false,
    "mergeable": null
  }
]
```

#### Status Codes

| Code | Description |
|------|-------------|
| `200 OK` | Successfully retrieved pull requests |
| `404 Not Found` | Repository does not exist or you do not have access |
| `422 Unprocessable Entity` | Invalid query parameter value |

---

### Get a Pull Request

Retrieve detailed information about a specific pull request, including its mergeable status, number of commits, and changed files.

**Method:** `GET`

**Path:** `/repos/{owner}/{repo}/pulls/{pull_number}`

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `owner` | string | Yes | The account owner of the repository |
| `repo` | string | Yes | The name of the repository |
| `pull_number` | integer | Yes | The number of the pull request |

#### Example Request

```bash
curl -H "Authorization: Bearer ghp_abc123def456" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/Uniswap/v4-core/pulls/1020
```

#### Example Response

```json
{
  "id": 1234567889,
  "number": 1020,
  "state": "open",
  "title": "docs: update README to reflect v4 mainnet deployment and improve structure",
  "user": {
    "login": "IAM-CW",
    "id": 12345678
  },
  "body": "This PR updates the v4-core README to reflect...",
  "created_at": "2026-03-18T09:00:00Z",
  "updated_at": "2026-03-18T09:00:00Z",
  "merged": false,
  "mergeable": true,
  "commits": 1,
  "additions": 45,
  "deletions": 22,
  "changed_files": 1,
  "html_url": "https://github.com/Uniswap/v4-core/pull/1020",
  "head": {
    "ref": "docs/readme-update",
    "sha": "a1b2c3d4e5f6"
  },
  "base": {
    "ref": "main",
    "sha": "f6e5d4c3b2a1"
  }
}
```

#### Status Codes

| Code | Description |
|------|-------------|
| `200 OK` | Successfully retrieved the pull request |
| `304 Not Modified` | Response has not changed since the last request (based on ETag or Last-Modified headers) |
| `404 Not Found` | Pull request does not exist or you do not have access |

---

### Create a Pull Request

Open a new pull request proposing changes from one branch to another.

**Method:** `POST`

**Path:** `/repos/{owner}/{repo}/pulls`

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `owner` | string | Yes | The account owner of the repository |
| `repo` | string | Yes | The name of the repository |

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | The title of the pull request |
| `body` | string | No | The description of the pull request. Supports Markdown |
| `head` | string | Yes | The branch containing your changes. For cross-repo PRs, use the format `username:branch` |
| `base` | string | Yes | The branch you want your changes pulled into |
| `draft` | boolean | No | If `true`, creates the pull request as a draft. Default: `false` |
| `maintainer_can_modify` | boolean | No | If `true`, allows maintainers of the base repository to push commits to the head branch. Default: `false` |

#### Example Request

```bash
curl -X POST \
  -H "Authorization: Bearer ghp_abc123def456" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/Uniswap/v4-core/pulls \
  -d '{
    "title": "fix: update outdated deployment addresses in README",
    "body": "## Summary\nThis PR corrects outdated contract addresses in the README.\n\n## Changes\n- Updated mainnet deployment address\n- Removed reference to deprecated testnet",
    "head": "IAM-CW:fix/readme-addresses",
    "base": "main",
    "draft": false
  }'
```

#### Example Response

```json
{
  "id": 1234567891,
  "number": 1035,
  "state": "open",
  "title": "fix: update outdated deployment addresses in README",
  "html_url": "https://github.com/Uniswap/v4-core/pull/1035",
  "created_at": "2026-04-04T12:00:00Z",
  "draft": false,
  "head": {
    "ref": "fix/readme-addresses",
    "label": "IAM-CW:fix/readme-addresses"
  },
  "base": {
    "ref": "main",
    "label": "Uniswap:main"
  }
}
```

#### Status Codes

| Code | Description |
|------|-------------|
| `201 Created` | Pull request successfully created |
| `403 Forbidden` | You do not have permission to create pull requests in this repository |
| `404 Not Found` | Repository does not exist or you do not have access |
| `422 Unprocessable Entity` | Validation error. Common causes: the head branch does not exist, the base branch does not exist, or a pull request already exists for this head/base combination |

---

### Update a Pull Request

Modify an existing pull request's title, body, state, or base branch.

**Method:** `PATCH`

**Path:** `/repos/{owner}/{repo}/pulls/{pull_number}`

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `owner` | string | Yes | The account owner of the repository |
| `repo` | string | Yes | The name of the repository |
| `pull_number` | integer | Yes | The number of the pull request |

#### Request Body

All fields are optional. Only include the fields you want to change.

| Field | Type | Description |
|-------|------|-------------|
| `title` | string | Updated title for the pull request |
| `body` | string | Updated description for the pull request |
| `state` | string | Change the state. Options: `open`, `closed` |
| `base` | string | Change the base branch the pull request targets |

#### Example Request

```bash
curl -X PATCH \
  -H "Authorization: Bearer ghp_abc123def456" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/Uniswap/v4-core/pulls/1020 \
  -d '{
    "title": "fix: update README to reflect v4 mainnet deployment",
    "body": "Updated PR description with additional context."
  }'
```

#### Example Response

```json
{
  "id": 1234567889,
  "number": 1020,
  "state": "open",
  "title": "fix: update README to reflect v4 mainnet deployment",
  "updated_at": "2026-04-04T12:30:00Z"
}
```

#### Status Codes

| Code | Description |
|------|-------------|
| `200 OK` | Pull request successfully updated |
| `403 Forbidden` | You do not have permission to update this pull request |
| `404 Not Found` | Pull request does not exist |
| `422 Unprocessable Entity` | Validation error (e.g., invalid base branch) |

---

### Merge a Pull Request

Merge an approved pull request into the base branch.

**Method:** `PUT`

**Path:** `/repos/{owner}/{repo}/pulls/{pull_number}/merge`

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `owner` | string | Yes | The account owner of the repository |
| `repo` | string | Yes | The name of the repository |
| `pull_number` | integer | Yes | The number of the pull request |

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `commit_title` | string | No | Title for the merge commit. Defaults to the PR title |
| `commit_message` | string | No | Additional detail for the merge commit. Defaults to the PR body |
| `merge_method` | string | No | The merge strategy. Options: `merge` (default), `squash`, `rebase` |
| `sha` | string | No | The SHA that the head must match to allow the merge. This prevents merging if the branch has been updated since you last reviewed it |

#### Example Request

```bash
curl -X PUT \
  -H "Authorization: Bearer ghp_abc123def456" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/Uniswap/v4-core/pulls/1020/merge \
  -d '{
    "commit_title": "fix: update README to reflect v4 mainnet deployment",
    "merge_method": "squash"
  }'
```

#### Example Response (Success)

```json
{
  "sha": "a1b2c3d4e5f6g7h8i9j0",
  "merged": true,
  "message": "Pull Request successfully merged"
}
```

#### Example Response (Cannot Merge)

```json
{
  "message": "Pull Request is not mergeable",
  "documentation_url": "https://docs.github.com/rest/pulls/pulls#merge-a-pull-request"
}
```

#### Status Codes

| Code | Description |
|------|-------------|
| `200 OK` | Pull request successfully merged |
| `403 Forbidden` | You do not have permission to merge this pull request |
| `404 Not Found` | Pull request does not exist |
| `405 Method Not Allowed` | Pull request is not mergeable (e.g., merge conflicts, required reviews not met, or status checks failing) |
| `409 Conflict` | A merge conflict exists that must be resolved before merging |

---

## Rate Limiting

The GitHub REST API enforces rate limits to prevent abuse.

| Authentication Level | Limit |
|---------------------|-------|
| Authenticated requests | 5,000 requests per hour |
| Unauthenticated requests | 60 requests per hour |

### Checking Your Rate Limit Status

Rate limit information is included in the response headers of every API request:

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Maximum number of requests allowed per hour |
| `X-RateLimit-Remaining` | Number of requests remaining in the current window |
| `X-RateLimit-Reset` | Unix timestamp (in seconds) when the rate limit resets |

#### Example

```bash
curl -I -H "Authorization: Bearer ghp_abc123def456" \
  https://api.github.com/repos/Uniswap/v4-core/pulls
```

Response headers:

```
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4987
X-RateLimit-Reset: 1712246400
```

If you exceed the rate limit, the API returns a `403 Forbidden` response with a message indicating when the limit resets.

---

## Common Error Responses

These error patterns apply across all endpoints:

### 401 Unauthorized

```json
{
  "message": "Bad credentials",
  "documentation_url": "https://docs.github.com/rest"
}
```

**Cause:** Missing, expired, or invalid authentication token. Generate a new token at [github.com/settings/tokens](https://github.com/settings/tokens).

### 403 Forbidden

```json
{
  "message": "API rate limit exceeded for user ID 12345678.",
  "documentation_url": "https://docs.github.com/rest/overview/rate-limits-for-the-rest-api"
}
```

**Cause:** You have exceeded the rate limit. Check the `X-RateLimit-Reset` header for when you can resume requests.

### 404 Not Found

```json
{
  "message": "Not Found",
  "documentation_url": "https://docs.github.com/rest"
}
```

**Cause:** The repository or pull request does not exist, or your token does not have permission to access it. Private repositories require the `repo` scope on your token.

### 422 Unprocessable Entity

```json
{
  "message": "Validation Failed",
  "errors": [
    {
      "resource": "PullRequest",
      "code": "custom",
      "message": "A pull request already exists for IAM-CW:docs/readme-update."
    }
  ]
}
```

**Cause:** The request body contains invalid data. The `errors` array provides specific details about what failed validation.

---

## Additional Resources

- [GitHub REST API official documentation](https://docs.github.com/en/rest)
- [GitHub authentication guide](https://docs.github.com/en/rest/authentication)
- [Creating a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- [GitHub API rate limiting](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api)
