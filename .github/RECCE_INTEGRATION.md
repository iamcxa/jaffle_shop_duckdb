# Recce MCP Integration with Claude Code

This document explains how to use Recce data validation tools with Claude Code in GitHub Actions.

## What is Recce?

Recce is a data validation toolkit that helps you compare dbt models between different environments (base vs current). It provides tools to check row counts, query differences, data profiles, and lineage changes.

## Available Recce Tools

When you mention `@claude` in a PR comment, Claude has access to the following Recce MCP tools:

### 1. `get_lineage_diff`
Get the lineage differences between base and current environments.

**Example usage:**
```
@claude 請使用 Recce 查看 lineage 的變化
```

### 2. `row_count_diff`
Compare row counts between base and current environments for specified models.

**Example usage:**
```
@claude 請使用 Recce 檢查 customers 和 orders 模型的 row count 差異
```

### 3. `query`
Execute a SQL query on the current or base environment.

**Example usage:**
```
@claude 使用 Recce 在 current 環境執行 query: select count(*) from {{ ref('customers') }}
```

### 4. `query_diff`
Execute queries on both environments and compare results.

**Example usage:**
```
@claude 使用 Recce 比較這個 query 在兩個環境的結果差異: select status, count(*) from {{ ref('orders') }} group by status
```

### 5. `profile_diff`
Generate and compare statistical profiles (min, max, avg, distinct count, etc.) for columns.

**Example usage:**
```
@claude 使用 Recce 比較 customers 模型的 profile 差異
```

## How It Works

When you mention `@claude` in a PR comment:

1. **Setup Phase** (runs automatically):
   - Checks out the Recce repository
   - Installs Python 3.10 and dependencies
   - Installs Recce from source
   - Generates dbt artifacts for base environment (cached)
   - Generates dbt artifacts for current environment
   - Validates MCP configuration

2. **Claude Execution**:
   - Claude receives your request
   - Uses Recce MCP tools to perform data validation
   - Returns results with analysis

## Requirements

- **dbt artifacts**: Both `target/` and `target-base/` directories must contain:
  - `manifest.json`
  - `catalog.json`
- **Database connection**: Ensure `profiles.yml` is correctly configured
- **GitHub Secrets**: `ANTHROPIC_API_KEY` must be set

## Workflow Configuration

The integration is configured in `.github/workflows/claude.yml`:

- **Triggers**: issue comments, PR review comments, issues (when containing `@claude`)
- **Caching**: 
  - Python dependencies
  - dbt packages
  - Base environment artifacts (keyed by branch + commit + project files)
- **Error Handling**: Graceful failures with warning messages
- **Validation**: Checks for artifact existence and MCP config validity

## Limitations

- **PR Context Only**: Recce tools are only available when running in a PR context
- **First Run**: Initial run may be slow as caches are being built
- **Database Access**: Requires valid database credentials in `profiles.yml`

## Troubleshooting

### Recce tools not available

**Symptoms**: Claude responds but cannot access Recce tools

**Solutions**:
1. Check that you're commenting on a PR (not an issue)
2. Verify the workflow ran successfully in the Actions tab
3. Check that base and current artifacts were generated
4. Look for validation warnings in workflow logs

### dbt artifacts not generated

**Symptoms**: Warning messages about missing artifacts

**Solutions**:
1. Check database connection configuration
2. Verify dbt project is valid (`dbt debug`)
3. Look at workflow logs for dbt command errors
4. Check if dbt packages are properly installed

### MCP configuration errors

**Symptoms**: MCP config validation fails

**Solutions**:
1. Verify `.github/mcp_config.json` is valid JSON
2. Check that Recce is properly installed in the workflow
3. Ensure environment variables are set correctly

## Performance Tips

- **Cache hits**: Most runs should use cached base artifacts, making them faster
- **Parallel execution**: Base and current artifact generation are optimized
- **Selective queries**: Be specific about which models to check

## Security Considerations

- **Source Code**: Recce is installed from the DataRecce/recce repository
- **Secrets**: Only `ANTHROPIC_API_KEY` is required
- **Database Credentials**: Stored in `profiles.yml`, ensure proper access controls
- **Permissions**: Workflow has read/write access to contents, issues, and PRs

## Example Workflows

### Check row counts for changed models
```
@claude 請使用 Recce 檢查所有有變更的模型的 row count
```

### Compare data profiles for a specific model
```
@claude 使用 Recce 比較 customers 模型的 email 和 customer_id 欄位的 profile
```

### Check lineage changes
```
@claude 使用 Recce 顯示這次 PR 的 lineage diff，特別是 downstream 的影響
```

### Custom query comparison
```
@claude 使用 Recce 比較這個 query: 
select date_trunc('month', order_date) as month, sum(amount) as total
from {{ ref('orders') }}
group by 1
order by 1
```

## Resources

- [Recce Documentation](https://docs.datarecce.io/)
- [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code)
- [MCP Protocol](https://modelcontextprotocol.io/)

