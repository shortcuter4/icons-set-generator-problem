# Icon Set Generator Problem

## Technical Stack
1. Spring Boot (core service implementation)
2. Postgres (to store icons metadata & operational records)
3. minIO (object storage e.g., icons, icon sets .svg, .png )
4. Liquibase etc...

## Initial plan (draft)
There will be 2 separate services. One service will be implemented to store icons uploaded by Product Designer. The other service will be implemented to successfully unique icon sets as an end goal. Postgres will be used to keep metadata of uploaded icons and their corresponding information like to which tags it is relating to etc. The other table will be used for operational purpose and to store unique sets of generated icon sets. And before each icons set generation it will be checked from there. minIO or some other alternatives (S3 etc) will be used to store .svg / .png etc icons and generated icons.   

## DB Design
- icons
  - it will be used as metadata for each uploaded icon by Product Designer for example
- tags
  - it will be used as tags dictionary
- icon_tags
  - it will be kept as a relation (many-to-many) between icons and tags
- icon_sets
  - there will be kept generated icon sets as an array of icon ids
- etc..

### Table and their corresponding fields
- icons
  -  id (PK)
  -  name
  -  tags[]
  -  file_path ---> minIO/S3 url (e.g., ://bucket/icons/1.svg)
  -  etc... (created_date, created_by, usage_count etc..)
-  tags
  - id (PK)
  - name
- icon_tags
  - icon_id
  - tag_id
- icon_sets
  - id
  - icon_ids[]
  - size
  - signature
  - signature_hash
  - img_file_url
  - etc...
- icon_sets_items
  - set_id
  - icon_id
 
#### Note on indexing strategies
1. Unique index will be used on icon_sets(signature_hash) - to avoid duplicates
2. GIN index will be used on icon_sets(icon_ids) - to speed up operations of checking overlap (to satisfy business rules like not overlapping 20% similarity rule etc.)
    - reference: https://www.postgresql.org/docs/current/gin.html
3. Lookup by name/category/tag:  B-tree indexes (icons, icon_tags)

### Icon generation approach - initial draft
1. It is possible to limit tags pickup using paramater like perTagLimit
2. Assuming we know overlap percentage and length/size of generated_icon_set that will be generated
   - we randomly pickup icons from tags pool (icons) up to given threshold and once we reach it then:
     - we pick one icon by one, and for each icon pickup we compare from icon_sets using their signature_hash (since all icons_ids are sorted when INSERT-ed into table)
     - we pick until the it satisfies the uniqueness and required length
     - then INSERT the resulting icon_sets into the table and generate image
     - then INSERT image to object storage and change the status to COMPLETED so that user will be able to download it

    
## Flow Diagrams
### Sequence flow
1. Case 1: Upload File Scenario 

  ![Upload File Scenario - Sequence Flow Diagram](upload-file.png)

2. Case 2: Generate Icon Set

  ![Generate Icon Set - Sequence Flow Diagram](generate-icons-set.png)

