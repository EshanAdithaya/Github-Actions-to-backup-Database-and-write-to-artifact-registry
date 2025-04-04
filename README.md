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










-----------------------------------------------------------------------------------------------------------------------------------------------------------------------







# MongoDB Daily Backup

This GitHub Actions workflow automatically creates daily backups of a specific MongoDB collection and stores them as GitHub artifacts.

## What This Workflow Does

This workflow performs the following operations:

1. **Scheduled Execution**: Runs daily at 4:30 AM UTC
2. **MongoDB Tools Installation**: Downloads and installs the MongoDB database tools
3. **Database Export**: Uses `mongoexport` to create a backup of the "jobs" collection from the "fcau" database
4. **Artifact Storage**: Saves the backup file as a GitHub artifact with a date-stamped name

## Workflow Breakdown

### Scheduling
```yaml
on:
  schedule:
    - cron: '30 4 * * *'
```
This section schedules the workflow to run at 4:30 AM UTC every day. The cron expression `'30 4 * * *'` represents: minute 30, hour 4, any day of month, any month, any day of week.

### Job Configuration
```yaml
jobs:
  backup:
    runs-on: ubuntu-latest
```
The workflow uses Ubuntu as the runner environment to perform the backup operations.

### Workflow Steps

#### 1. Checkout Repository
```yaml
- name: Checkout Repo
  uses: actions/checkout@v3
```
This step checks out your repository code to the runner. This is necessary for the workflow to access any scripts or configurations in your repository.

#### 2. Install MongoDB Tools
```yaml
- name: Install MongoDB Tools
  run: |
    wget https://fastdl.mongodb.org/tools/db/mongodb-database-tools-ubuntu2004-x86_64-100.9.0.tgz
    tar -xvzf mongodb-database-tools-*.tgz
    sudo mv mongodb-database-tools-*/bin/* /usr/local/bin/
```
This step:
- Downloads the MongoDB database tools package (version 100.9.0 for Ubuntu 20.04)
- Extracts the tools from the downloaded archive
- Moves the executable tools to a directory in the system PATH for easy access

#### 3. Run MongoDB Backup
```yaml
- name: Run MongoDB Backup
  run: |
    mongoexport --uri="${{ secrets.MONGO_URI }}" --db="fcau" --collection="jobs" --out=backup_$(date +%Y-%m-%d).json --jsonArray
```
This step:
- Uses `mongoexport` to connect to your MongoDB instance using the connection string stored in the GitHub secret `MONGO_URI`
- Targets the "jobs" collection in the "fcau" database
- Exports the data to a JSON file with today's date in the filename (format: backup_YYYY-MM-DD.json)
- Uses the `--jsonArray` flag to format the output as a JSON array

#### 4. Set Artifact Name
```yaml
- name: Set Artifact Name
  id: set-artifact-name
  run: echo "ARTIFACT_NAME=mongo_backup_$(date +%Y-%m-%d)" >> $GITHUB_ENV
```
This step:
- Creates an environment variable called `ARTIFACT_NAME` with a value that includes today's date
- The format is "mongo_backup_YYYY-MM-DD"
- This variable will be used to name the GitHub artifact

#### 5. Upload Backup as Artifact
```yaml
- name: Upload to GitHub Artifacts
  uses: actions/upload-artifact@v4
  with:
    name: ${{ env.ARTIFACT_NAME }}
    path: backup_*
```
This step:
- Uses the GitHub Actions upload-artifact@v4 action
- Names the artifact using the environment variable set in the previous step
- Uploads all files that match the pattern "backup_*" (which will include our dated backup file)

## Setup Instructions

### 1. Add the Workflow File

Place this workflow file in your repository under `.github/workflows/mongodb-daily-backup.yml`.

### 2. Configure GitHub Secret

You must set up the following secret in your GitHub repository:

| Secret Name | Description |
|-------------|-------------|
| `MONGO_URI` | MongoDB connection string with authorization credentials |

The MongoDB URI typically follows this format:
```
mongodb+srv://username:password@hostname/database?options
```

To add this secret:
1. Go to your GitHub repository
2. Click on "Settings" > "Secrets and variables" > "Actions"
3. Click "New repository secret"
4. Name: `MONGO_URI`
5. Value: Your MongoDB connection string

### 3. Accessing Backup Files

Backup files are stored as GitHub artifacts:
1. Go to your GitHub repository
2. Click on "Actions"
3. Find and click on the completed workflow run
4. Scroll down to the "Artifacts" section
5. Download the artifact named `mongo_backup_YYYY-MM-DD`

## Customization Options

### Backing Up Different Collections

To back up a different collection, modify the `mongoexport` command:

```yaml
mongoexport --uri="${{ secrets.MONGO_URI }}" --db="your_database" --collection="your_collection" --out=backup_$(date +%Y-%m-%d).json --jsonArray
```

### Changing the Backup Format

For BSON format instead of JSON:

```yaml
mongodump --uri="${{ secrets.MONGO_URI }}" --db="fcau" --collection="jobs" --out=backup_$(date +%Y-%m-%d)
```

### Changing the Backup Schedule

Modify the cron expression to change when the backup runs:

```yaml
on:
  schedule:
    - cron: '0 0 * * *'  # Runs at midnight UTC daily
```

## Important Notes

- GitHub artifacts are automatically deleted after 90 days
- For long-term storage, consider modifying the workflow to push backups to external storage
- The MongoDB connection string contains sensitive credentials, so it must be stored as a secret

- The MySQL password is stored as a GitHub Secret and is masked in logs
- Consider using a dedicated backup user with minimal permissions (SELECT, LOCK TABLES, SHOW VIEW, TRIGGER)
- Avoid storing sensitive data in GitHub artifacts for extended periods
