# MySQL Daily Backup

This GitHub Actions workflow automatically creates daily backups of your MySQL database and stores them as GitHub artifacts.

## What This Workflow Does

1. **Scheduled Execution**: Runs daily at 4:30 AM UTC
2. **Database Export**: Uses `mysqldump` to create a complete backup of your specified MySQL database(s)
3. **Artifact Storage**: Saves the backup file as a GitHub artifact with a date-stamped name

## Setup Instructions

### 1. Copy the Workflow File

Place the `mysql-daily-backup.yml` file in your repository under the `.github/workflows/` directory.

### 2. Configure GitHub Secrets

You must set up the following secrets in your GitHub repository:

| Secret Name | Description |
|-------------|-------------|
| `MYSQL_HOST` | The hostname or IP address of your MySQL server |
| `MYSQL_USER` | MySQL username with sufficient privileges to perform backups |
| `MYSQL_PASSWORD` | Password for the MySQL user |
| `MYSQL_DATABASE` | Name of the database to back up |

To add these secrets:
1. Go to your GitHub repository
2. Click on "Settings" > "Secrets and variables" > "Actions"
3. Click "New repository secret" for each secret you need to add

### 3. Customization Options

#### Backup Schedule

The default schedule is set to run daily at 4:30 AM UTC. You can modify the cron expression in the workflow file to change this schedule:

```yaml
on:
  schedule:
    - cron: '30 4 * * *'  # Format: minute hour day-of-month month day-of-week
```

#### Backing Up Multiple Databases

To back up multiple databases, modify the `mysqldump` command in the workflow:

```yaml
# For specific databases
mysqldump \
  --host=${{ secrets.MYSQL_HOST }} \
  --user=${{ secrets.MYSQL_USER }} \
  --password=${{ secrets.MYSQL_PASSWORD }} \
  --databases db1 db2 db3 \
  --result-file=backups/backup_$(date +%Y-%m-%d).sql

# OR, to back up all databases
mysqldump \
  --host=${{ secrets.MYSQL_HOST }} \
  --user=${{ secrets.MYSQL_USER }} \
  --password=${{ secrets.MYSQL_PASSWORD }} \
  --all-databases \
  --result-file=backups/backup_$(date +%Y-%m-%d).sql
```

#### Additional mysqldump Options

You can add more options to the `mysqldump` command for specialized backups:

- `--no-data`: Structure only, no data
- `--no-create-info`: Data only, no table structures
- `--events --routines --triggers`: Include stored procedures, functions, and triggers

## Managing Backups

### Accessing Backup Files

1. Go to your GitHub repository
2. Click on "Actions"
3. Find and click on the completed workflow run
4. Scroll down to the "Artifacts" section
5. Download the artifact named `mysql_backup_YYYY-MM-DD`

### Retention Period

GitHub artifacts are automatically deleted after 90 days. If you need longer retention, consider:

1. Setting up a workflow step to push backups to an external storage service (AWS S3, Google Cloud Storage, etc.)
2. Downloading important backups manually and storing them securely

## Troubleshooting

- **Connection Issues**: Verify that your MySQL server allows connections from GitHub Actions IPs
- **Permission Errors**: Ensure the MySQL user has sufficient privileges for backup operations
- **Large Databases**: For databases >2GB, consider compressing the output or using a more robust storage solution

## Security Considerations

- The MySQL password is stored as a GitHub Secret and is masked in logs
- Consider using a dedicated backup user with minimal permissions (SELECT, LOCK TABLES, SHOW VIEW, TRIGGER)
- Avoid storing sensitive data in GitHub artifacts for extended periods
