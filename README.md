# APort Policy Verify Action

Verify pull requests against APort agent policies to ensure compliance and security.

## 🚀 Features

- **Policy Enforcement** - Verify PRs against APort agent policies
- **GitHub Integration** - Seamless integration with GitHub Actions
- **Context Mapping** - Automatic mapping of GitHub context to APort policies
- **Flexible Configuration** - Customizable policy packs and enforcement rules
- **PR Comments** - Automatic PR comments with policy results
- **Multiple Triggers** - Support for various GitHub events

## 📋 Usage

### Basic Usage

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
          fail-on-violation: true
          comment-on-pr: true
```

### Advanced Usage

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
          fail-on-violation: true
          comment-on-pr: true
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## 🔧 Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `agent-id` | The APort Agent ID to use for policy verification | ✅ | - |
| `policy-pack` | The APort policy pack ID to enforce (e.g., code.repository.merge.v1) | ✅ | `code.repository.merge.v1` |
| `api-base` | The base URL for the APort API | ❌ | `https://api.aport.io` |
| `fail-on-violation` | Whether the action should fail if a policy violation is detected | ❌ | `true` |
| `comment-on-pr` | Whether to add a comment to the pull request with policy results | ❌ | `true` |
| `github-token` | GitHub token for commenting on PRs | ❌ | `${{ github.token }}` |

## 📤 Outputs

| Output | Description |
|--------|-------------|
| `allow` | Boolean indicating if the action is allowed by policy |
| `reasons` | JSON array of reasons for policy violation, if any |

## 🏗️ Setup

### 1. Create APort Agent

1. Go to [APort Dashboard](https://aport.io)
2. Create a new agent
3. Configure policy settings
4. Note the Agent ID

### 2. Set GitHub Secrets

Add the following secrets to your repository:

```bash
# Required
APORT_AGENT_ID=your-agent-id-here

# Optional (for PR comments)
GITHUB_TOKEN=your-github-token
```

### 3. Configure Agent Passport

In your APort agent passport, set up the following:

```json
{
  "integrations": {
    "github": {
      "allowed_actors": ["your-bot[bot]", "acme-ci"],
      "allowed_apps": ["your-github-app"]
    }
  },
  "capabilities": ["code.repository.merge.v1"],
  "assurance_level": 3
}
```

## 📊 Policy Packs

### code.repository.merge.v1 - Repository Safety

Enforces comprehensive repository-level safety policies:

**Required Capabilities:**
- `repo.pr.create` - Create pull requests
- `repo.merge` - Merge pull requests

**Required Limits:**
- `max_prs_per_day` - Daily PR creation limit
- `max_merges_per_day` - Daily merge limit
- `max_pr_size_kb` - Maximum PR size in KB

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
| `github.repository` | `repo` | Repository name |
| `github.event.pull_request.base.ref` | `base_branch` | Target branch |
| `github.actor` | `github_actor` | GitHub actor |
| `github.app` | `github_app` | GitHub app |
| Computed diff stats | `files_changed`, `lines_added` | Change statistics |
| PR size calculation | `pr_size_kb` | PR size in KB for size limits |
| Changed file paths | `file_paths` | File paths for path allowlist enforcement |
| Review requirements | `requires_review` | Whether review is required |
| PR labels | `labels` | Pull request labels |
| Review count | `reviews` | Number of reviews |
| Draft status | `is_draft` | Whether PR is draft |
| Mergeable status | `is_mergeable` | Whether PR can be merged |

## 🚨 Troubleshooting

### Common Issues

1. **Agent not found**
   - Verify `APORT_AGENT_ID` is correct
   - Check agent exists in APort dashboard

2. **Policy violations**
   - Review agent passport configuration
   - Check policy pack requirements
   - Verify GitHub actor/app permissions

3. **API errors**
   - Verify `api-base` URL is correct
   - Check network connectivity
   - Review API rate limits

### Debug Mode

Enable debug logging:

```yaml
- name: APort Policy Verification
  uses: aporthq/policy-verify-action@v1
  with:
    agent-id: ${{ secrets.APORT_AGENT_ID }}
    debug: true
```

## 📚 Examples

See the `example-repo/` directory for complete working examples.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📄 License

MIT License - see LICENSE file for details.

## 🆘 Support

- 📖 [Documentation](https://aport.io/docs)
- 💬 [Discord Community](https://discord.gg/aport)
- 🐛 [Issue Tracker](https://github.com/aporthq/policy-verify-action/issues)
- 📧 [Email Support](mailto:support@aport.io)