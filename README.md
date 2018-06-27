<!-- .slide: class="center" -->
# Migrate Training
## + Hands-on lab

 - Lucas Hedding (heddn)

Formatted as slides here:
https://mtechllc.github.io/migrate-lab/#/



<!-- .slide: class="align-left heading-space -->
## What is migrate?

 - A method to import content and configuration
     - Configuration from an old D6 site
     - Term data from CSV
     - Hierarchy of data from XML
     - Etc.



<!-- .slide: class="align-left heading-space -->
## Historical Note

 - Forget what you know about migrate for Drupal 7
 - Migrate in core is a pluggable Extract, Transform, Load (ETL) process



<!-- .slide: class="align-left heading-space -->
## What is migrate?

 - A method to extract content and configuration, transform it and load into Drupal 8
     - Soure plugin (extract)
     - Process pluggin (transform)
     - Destination plugin (load)



<!-- .slide: class="align-left heading-space -->
## What is migrate?
### Source Plugin
 - Extract data from somewhere
 - (optionally) accepts some configuration to locate the data and/or format it
 - Examples:
    - SQL
    - CSV
    - XML
    - JSON

Note:
SQL is in core, the rest are in contrib.



<!-- .slide: class="align-left heading-space -->
## What is migrate?
### Process Plugin
 - Transform data
 - (optionally) accepts some configuration
 - Examples:
    - get
    - explode
    - concat
    - callback
    - entity_lookup or entity_generate
 
Note:
 - The full list of re-usable plugins in core (and several in contrib) can be found at: https://www.drupal.org/docs/8/api/migrate-api/migrate-process/migrate-process-overview
- The last one, plus several others are in Migrate Plus



<!-- .slide: class="align-left heading-space -->
## What is migrate?
### Destination Plugin
 - Load data into the destination
 - (optionally) accepts some configuration
 - Examples:
   - EntityContentBase
   - EntityConfigBase
   - EntityReferenceRevisions
   
Note:
 - Node, terms, users, etc are content.
- Field instances, field base, strongarm variables, etc are configuration.
- Almost all of these exist in core. The lone none exception is for Paragraphs, which stores its data in entity_reference_revisions.
- See https://www.mtech-llc.com/blog/charlotte-leon/migration-csv-data-paragraphs for a full write-up on how to migrate into Paragraphs.



<!-- .slide: class="align-left heading-space -->
## Modules
### Core
  - migrate
  - migrate_drupal
  - migrate_drupal_ui

Note:
  - Only migrate is needed if you aren't importing data from a previous Drupal site.
- Only migrate_drupal is needed if you plan to import using drush.



<!-- .slide: class="align-left heading-space -->
## Modules
### Contrib
  - migrate_plus
    - Store migration as config entities
    - Import from JSON / XML
    - Several non-core but still useful process plugins
  - migrate_tools
    - Run migrations from command line (drush)
    - Run migrations from web UI
  - migrate_source_csv
    - Import CSV data
  - wordpress_migrate
  - commerce_migrate

Note:
 - Other source plugins exist in contrib, but these are the most popular.
- Of other note for running migrations: migrate_manifest, migrate_run



<!-- .slide: class="align-left" -->
## Workflow

 - Create the config
 - Import the config
 - Run the config
 - Modify the config
 - Import the new config
 - Run the config
 - Rinse & repeat as necessary



<!-- .slide: class="align-left" -->
## Migration example
    # migrate_plus.migration.omdb_json.yml
    id: omdb_json
    label: 'JSON feed of movies'
    source:
      plugin: url
      data_fetcher_plugin: http
      data_parser_plugin: json
      include_raw_data: true
      urls: 'http://www.omdbapi.com/?s=space&type=movie&r=json&apikey=86e4b169'
      item_selector: Search
      fields:
        -
          name: title
          label: Title
          selector: Title
        -
          name: year
          label: 'Publish Year'
          selector: Year
        -
          name: imdbID
          label: 'IMDB ID'
          selector: imdbID
        -
          name: poster
          label: 'Image URL to a movie poster'
          selector: Poster
      ids:
        imdbID:
          type: string
      constants:
        bundle: movie
        day_month: 01-01-
        destination_directory: 'public://posters/'
        file_extension: .jpg
    process:
      type: constants/bundle
      title: title
      field_published_date:
        -
          plugin: concat
          source:
            - constants/day_month
            - year
        -
          plugin: format_date
          from_format: d-m-Y
          to_format: Y-m-d
          timezone: America/Managua
      destination_basename:
        plugin: callback
        callable: basename
        source: poster
      destination_path:
        plugin: concat
        source:
          - constants/destination_directory
          - '@destination_basename'
      field_image/target_id:
        -
          plugin: file_copy
          source:
            - poster
            - '@destination_path'
        -
          plugin: entity_generate
          value_key: uri
          entity_type: file
      field_image/alt: title
      field_image/title: title
    destination:
      plugin: 'entity:node'



<!-- .slide: class="align-left" -->
## Process Chain

Implicit get plugin:

    process:
      title: subject
   
Specific process plugin: 

    process:
      field_names:
        plugin: explode
        migration: ','
        source: authors




<!-- .slide: class="align-left" -->
## Process Chain

Chained process: 

      field_full_name:
        -
          plugin: concat
          source:
            - first_name
            - last_name
          delimiter: ' '



<!-- .slide: class="align-left" -->
## Workflow: run the migrations

Import your configuration if you made changes.

    drush config-import
    
Flush cache

    drush cr

Run all migrations!

    drush migrate-import --all

Or run only a single migration:

    drush migrate-import omdb_json



<!-- .slide: class="align-left" -->
## Further Reading

 - [Custom Drupal-to-Drupal Migrations with Migrate Tools](https://drupalize.me/blog/201605/custom-drupal-drupal-migrations-migrate-tools)
 - [Categorizing Migrations According to Their Type](https://www.mtech-llc.com/blog/edys-meza/categorizing-migrations-according-their-type)
 - [Config Management & Migrations](https://www.mtech-llc.com/blog/lucas-hedding/config-management-migrations)



<!-- .slide: class="align-left" -->
## Module for Migration Configuration
### Module Structure


    $ tree
    .
    ├── config
    │   └── install
    │       ├── migrate_plus.migration_group.default.yml
    │       └── migrate_plus.migration.omdb_json.yml
    ├── custom_migrate.info.yml




<!-- .slide: class="align-left" -->
## JSON example

    {
      "Title": "Plan 9 from Outer Space",
      "Year": "1959",
      "imdbID": "tt0052077",
      "Type": "movie",
      "Poster": "https:\/\/images-na.ssl-images-amazon.com\/images\/M\/MV5BMzUzMzA0NDE3MF5BMl5BanBnXkFtZTgwMzg1Mjc1MDE@._V1_SX300.jpg"
    }




<!-- .slide: class="align-left" -->
## Manual method, after enabling

    # drush ms                                             
    Group: default  Status  Total  Imported  Unprocessed  Last imported 
    omdb_json         Idle    10      0         10  

Now import!

    # drush mi omdb_json
    Processed 4 items (10 created, 0 updated, 0 failed, 0 ignored) - done    [status]
    with 'omdb_json'



<!-- .slide: class="align-left" -->
## Manual Method: Further Reading

 - [Drupal 6 to Drupal 8(.1.x) Custom Content Migration](https://www.drupaleasy.com/blogs/ultimike/2016/04/drupal-6-drupal-81x-custom-content-migration)
 - [Migrating Date Ranges into Date Range Module](https://www.mtech-llc.com/blog/gerardo-hernandez/migrating-date-ranges-csv-date-range-module)
 - [Migration of Data into Paragraphs](https://www.mtech-llc.com/blog/charlotte-leon/migration-csv-data-paragraphs)
 - [How to Migrate Images](https://www.mtech-llc.com/blog/ada-hernandez/how-migrate-images-drupal-8-using-csv-source)



<!-- .slide: class="align-left" -->
## Hands-on lab

- Together!



<!-- .slide: class="heading-space -->
- Lucas Hedding (heddn) [@lucashedding](https://twitter.com/lucashedding)
 <a href="https://www.mtech-llc.com"><img style="vertical-align: middle;" height=80px src=images/mtech.svg></a>

Slidedeck liberaly inspired by [@ryan_weal](https://twitter.com/ryan_weal)'s [Migrate Shotgun Tour](https://github.com/kafeiinteractif/shotgun-migrate-tour) presentation.
