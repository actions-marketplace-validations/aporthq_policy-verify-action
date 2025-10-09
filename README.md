# APort Policy Verification Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-APort%20Policy%20Verify-blue.svg?logo=github)](https://github.com/marketplace/actions/aport-policy-verify)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/aporthq/policy-verify-action)

Verify pull requests against APort agent policies to ensure compliance, security, and proper authorization for automated actions.

## 🚀 Features

- **🛡️ Policy Enforcement** - Verify PRs against APort agent policies with comprehensive context mapping
- **🔗 GitHub Integration** - Seamless integration with GitHub Actions and PR workflows
- **📊 Rich Context** - Automatic mapping of GitHub context to APort policy requirements
- **⚙️ Flexible Configuration** - Customizable policy packs, timeouts, and enforcement rules
- **💬 PR Comments** - Automatic PR comments with detailed policy results and next steps
- **🔄 Retry Logic** - Built-in retry mechanism with exponential backoff for reliability
- **🔐 Authentication** - Support for API key authentication and GitHub token integration
- **📈 Multiple Triggers** - Support for various GitHub events and manual dispatch

## 📋 Quick Start

### 1. Basic Usage

```yaml
name: APort Policy Check
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  policy-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: APort Policy Verification
        uses: aporthq/policy-verify-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          policy-pack: 'code.repository.merge.v1'
```

### 2. Advanced Usage

```yaml
name: Advanced APort Policy Check
on:
  pull_request:
    types: [opened, synchronize, labeled, ready_for_review]
  workflow_dispatch:
    inputs:
      agent_id:
        description: 'Agent ID to verify'
        required: true

jobs:
  policy-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: APort Policy Verification
        uses: aporthq/policy-verify-action@v1
        with:
          agent-id: ${{ github.event.inputs.agent_id || secrets.APORT_AGENT_ID }}
          policy-pack: 'code.repository.merge.v1'
          api-base: 'https://api.aport.io'
          api-key: ${{ secrets.APORT_API_KEY }}
          fail-on-violation: true
          comment-on-pr: true
          timeout-seconds: 30
```

## 🔧 Inputs

| Input | Description | Required | Default | Example |
|-------|-------------|----------|---------|---------|
| `agent-id` | The APort Agent ID to use for policy verification | ✅* | - | `ap_1234567890abcdef` |
| `policy-pack` | The APort policy pack ID to enforce | ❌ | `code.repository.merge.v1` | `code.repository.merge.v1` |
| `api-base` | The base URL for the APort API | ❌ | `https://api.aport.io` | `https://api.aport.io` |
| `api-key` | APort API key for authenticated requests | ❌ | - | `aport_sk_...` |
| `fail-on-violation` | Fail the workflow if policy is violated | ❌ | `true` | `true` |
| `comment-on-pr` | Add comment to PR with verification results | ❌ | `true` | `true` |
| `timeout-seconds` | Request timeout in seconds | ❌ | `30` | `60` |

*Required if `APORT_AGENT_ID` secret is not set

## 📤 Outputs

| Output | Description | Type |
|--------|-------------|------|
| `allowed` | Whether the agent is allowed to proceed | `boolean` |
| `reasons` | JSON array of reasons for policy violation, if any | `string` |

## 🏗️ Setup Guide

### 1. Create APort Agent

1. Go to [APort Dashboard](https://aport.io)
2. Create a new agent
3. Configure policy settings and capabilities
4. Note the Agent ID (format: `ap_xxxxxxxxx`)

### 2. Configure GitHub Secrets

Add the following secrets to your repository:

```bash
# Required
APORT_AGENT_ID=ap_1234567890abcdef

# Optional (for API authentication)
APORT_API_KEY=aport_sk_...

# Optional (for PR comments)
GITHUB_TOKEN=ghp_...
```

### 3. Configure Agent Passport

In your APort agent passport, configure the following:

```json
{
  "name": "My GitHub Bot",
  "capabilities": ["code.repository.merge.v1"],
  "assurance_level": 3,
  "integrations": {
    "github": {
      "allowed_actors": ["my-bot[bot]", "acme-ci"],
      "allowed_apps": ["my-github-app"]
    }
  },
  "limits": {
    "max_files_changed": 100,
    "max_lines_added": 1000,
    "max_pr_size_kb": 500,
    "required_reviews": 1
  }
}
```

## 📊 Policy Packs

### code.repository.merge.v1 - Repository Safety

Enforces comprehensive repository-level safety policies for pull request operations.

**Required Capabilities:**
- `repo.pr.create` - Create pull requests
- `repo.merge` - Merge pull requests
- `repo.branch.create` - Create branches
- `repo.branch.delete` - Delete branches

**Enforcement Rules:**
- **Repository Allowlist** - Restrict which repositories the agent can act on
- **Base Branch Allowlist** - Control which branches can receive PRs
- **Path Allowlist** - Restrict file access patterns
- **Size Limits** - Prevent oversized PRs and file uploads
- **Review Requirements** - Enforce code review policies
- **GitHub Actor Validation** - Verify the GitHub actor is authorized
- **GitHub App Validation** - Verify the GitHub app is authorized

**Minimum Assurance Level:** L2 (Level 2)

## 🔍 Context Mapping

The action automatically maps GitHub context to APort policy context:

| GitHub Context | APort Context | Description |
|----------------|---------------|-------------|
| `github.repository` | `repository` | Repository name (org/repo) |
| `github.event.pull_request.base.ref` | `branch` | Target branch |
| `github.event.pull_request.base.ref` | `base_branch` | Base branch for comparison |
| `github.actor` | `github_actor` | GitHub actor performing the action |
| `github.app` | `github_app` | GitHub app (if applicable) |
| Computed diff stats | `files_changed`, `lines_added`, `lines_removed` | Change statistics |
| PR size calculation | `pr_size_kb` | PR size in KB for size limits |
| Changed file paths | `file_paths` | File paths for path allowlist enforcement |
| Review requirements | `requires_review` | Whether review is required |
| PR labels | `labels` | Pull request labels array |
| Review count | `reviews` | Number of reviews |
| Draft status | `is_draft` | Whether PR is draft |
| Mergeable status | `is_mergeable` | Whether PR can be merged |
| PR title | `title` | Pull request title |
| PR description | `description` | Pull request body/description |

## 🚨 Troubleshooting

### Common Issues

1. **"Agent not found" error**
   - Verify `APORT_AGENT_ID` is correct and exists in APort dashboard
   - Check agent is active and not expired
   - Ensure agent ID format is `ap_xxxxxxxxx`

2. **"Policy violations" error**
   - Review agent passport configuration
   - Check if GitHub actor is in `allowed_actors` list
   - Verify GitHub app is in `allowed_apps` list
   - Check policy limits are appropriate for your use case

3. **"API errors" error**
   - Verify `api-base` URL is correct and accessible
   - Check network connectivity and firewall settings
   - Review API rate limits and authentication
   - Ensure API key is valid (if using authentication)

4. **"Invalid JSON response" error**
   - Check API endpoint is returning valid JSON
   - Verify API version compatibility
   - Review request payload format

### Debug Mode

Enable debug logging by setting the `DEBUG` environment variable:

```yaml
- name: APort Policy Verification
  uses: aporthq/policy-verify-action@v1
  env:
    DEBUG: true
  with:
    agent-id: ${{ secrets.APORT_AGENT_ID }}
```

### Testing Locally

Use the provided test scripts for local development:

```bash
# Test API directly
./test-api.sh

# Test with local server
./test-local.sh

# Test with mock server
node mock-api-server.js
```

## 📚 Examples

### Basic Repository Protection

```yaml
name: Repository Protection
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  protect:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aporthq/policy-verify-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          policy-pack: 'code.repository.merge.v1'
          fail-on-violation: true
          comment-on-pr: true
```

### Multi-Policy Verification

```yaml
name: Multi-Policy Check
on: [pull_request]

jobs:
  security-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aporthq/policy-verify-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          policy-pack: 'code.repository.merge.v1'
          comment-on-pr: false

  compliance-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aporthq/policy-verify-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          policy-pack: 'code.repository.merge.v1'
          comment-on-pr: true
```

### Conditional Verification

```yaml
name: Conditional Policy Check
on: [pull_request]

jobs:
  policy-check:
    runs-on: ubuntu-latest
    if: github.actor != 'dependabot[bot]'
    steps:
      - uses: actions/checkout@v4
      - uses: aporthq/policy-verify-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          policy-pack: 'code.repository.merge.v1'
```

### Manual Dispatch

```yaml
name: Manual Policy Check
on:
  workflow_dispatch:
    inputs:
      agent_id:
        description: 'Agent ID to verify'
        required: true
        type: string

jobs:
  manual-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aporthq/policy-verify-action@v1
        with:
          agent-id: ${{ github.event.inputs.agent_id }}
          policy-pack: 'code.repository.merge.v1'
```

## 🔒 Security

- **API Key Protection** - API keys are masked in logs and stored securely
- **Input Validation** - All inputs are validated before processing
- **Error Handling** - Sensitive information is not exposed in error messages
- **Rate Limiting** - Built-in retry logic prevents API abuse
- **Audit Trail** - All policy decisions are logged with request IDs

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- 📖 [APort Documentation](https://docs.aport.io)
- 💬 [Discord Community](https://discord.gg/aport)
- 🐛 [Issue Tracker](https://github.com/aporthq/policy-verify-action/issues)
- 📧 [Email Support](mailto:support@aport.io)

## 🔗 Related

- [APort Dashboard](https://aport.io)
- [APort API Documentation](https://docs.aport.io/api)
- [Policy Packs Documentation](https://docs.aport.io/policies)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

---

**Made with ❤️ by the APort Team**

*APort Policy Verify Action v1.0.0 - Production Ready*