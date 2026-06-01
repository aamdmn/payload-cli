# Common Patterns & Recipes

## Content Creation Pipeline

```bash
# 1. Discover and understand
payload-cli collections
payload-cli describe posts

# 2. Create draft content
payload-cli create posts --data '{"title":"New Article","slug":"new-article","status":"draft"}'

# 3. Verify creation
payload-cli find posts --where '{"slug":{"equals":"new-article"}}' --select '{"id":true,"title":true,"status":true}'

# 4. Publish when ready
payload-cli update posts <id> --data '{"status":"published"}'
```

## Bulk Status Update

```bash
# Preview what will change
payload-cli find posts --where '{"status":{"equals":"draft"}}' --select '{"id":true,"title":true}'

# Dry run the update
payload-cli update-many posts --where '{"status":{"equals":"draft"}}' --data '{"status":"published"}' --dry-run

# Execute
payload-cli update-many posts --where '{"status":{"equals":"draft"}}' --data '{"status":"published"}'
```

## Search and Modify

```bash
# Find documents matching criteria
payload-cli find posts --where '{"title":{"contains":"old keyword"}}' --json

# Update each one (or use update-many for uniform changes)
payload-cli update posts <id1> --data '{"title":"New Title 1"}'
payload-cli update posts <id2> --data '{"title":"New Title 2"}'
```

## Safe Cleanup

```bash
# Step 1: Preview what will be deleted
payload-cli delete-many posts --where '{"status":{"equals":"archived"}}'

# Step 2: Verify the list is correct
# Step 3: Confirm deletion
payload-cli delete-many posts --where '{"status":{"equals":"archived"}}' --confirm
```

## Working with Globals

```bash
# Read current settings
payload-cli get-global site-settings

# Update a specific field
payload-cli update-global site-settings --data '{"maintenanceMode":true}'

# Verify
payload-cli get-global site-settings --select '{"maintenanceMode":true}'
```

## JSON Output for Scripting

```bash
# Get document IDs as JSON
payload-cli find posts --select '{"id":true}' --json | jq '.[].id'

# Count documents
payload-cli find posts --where '{"status":{"equals":"published"}}' --json | jq '.totalDocs'

# Chain commands
ID=$(payload-cli create posts --data '{"title":"Test"}' --json | jq -r '.id')
payload-cli find-by-id posts "$ID"
```

## Media Upload Pipeline

```bash
# 1. Check which collections support uploads
payload-cli collections

# 2. Understand the media schema (alt text, etc.)
payload-cli describe media

# 3. Upload a single file
payload-cli upload media ./hero.jpg --data '{"alt":"Hero banner image"}'

# 4. Bulk upload a directory
payload-cli upload media ./photos/

# 5. Verify uploads
payload-cli find media --limit 5 --select '{"id":true,"filename":true,"alt":true}'
```

## Creating Content with Images

```bash
# Describe the target collection to find upload fields
payload-cli describe pages

# Create a page with an auto-uploaded hero image
payload-cli create pages --data '{"title":"About Us","slug":"about"}' --file 'heroImage=./hero.jpg'

# Create with multiple file fields
payload-cli create pages --data '{"title":"Gallery"}' --file 'heroImage=./hero.jpg' --file 'thumbnail=./thumb.png'

# Upload separately, then reference by ID
payload-cli upload media ./hero.jpg --json   # Get the ID from output
payload-cli create pages --data '{"title":"About","heroImage":"<media-id>"}'
```

## Download Media

```bash
# Download a specific file
payload-cli download media <id> --out ./downloads/

# Download all images matching a query
payload-cli download media --where '{"mimeType":{"contains":"image"}}' --out ./backups/

# Dry run to see what would be downloaded
payload-cli download media --where '{"alt":{"contains":"hero"}}' --dry-run
```

## Data Inspection

```bash
# Quick overview of a collection
payload-cli find posts --limit 3 --select '{"id":true,"title":true,"status":true,"createdAt":true}'

# Full document inspection
payload-cli find-by-id posts <id> --json --depth 2

# Check relationships
payload-cli find posts --where '{"author":{"equals":"<user-id>"}}' --select '{"title":true}'
```
