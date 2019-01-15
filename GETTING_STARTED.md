The mds-provider-services repo on Github is a collection of services that can be used to work with MDS provider data. A majority of these services are direct clients to the mds-provider repo, which is essentially a library containing tools to work with MDS provider data.

# Ingesting data and using the availability calculation

All commands should be run in the root of the repository unless noted otherwise

### Setting up the repo

1. Run `cp docker-compose.dev.yml docker-compose.yml`   
2. Run `cp .env.sample .env`    to create your environment config

### Initializing the database

1. Run `bin/initdb.sh` to initialize the database
    a. It’s important to read this script to fully understand what this is doing. As of [this PR](https://github.com/CityofSantaMonica/mds-provider-services/pull/23/files#diff-8439bb9410daacb29a7eae6e1129410f), this script resets the database by dropping status_changes and trips tables, and recreates them.
2. Connect pgadmin to your database by going to `http://localhost:PGADMIN_HOST_PORT` and login with `PGADMIN_DEFAULT_EMAIL` and `PGADMIN_DEFAULT_PASSWORD` as defined in your `.env` file
    a. Then right click on "Servers" on the left, click "create -> server", give it any name in the “General” tab, and then in the “Connection” tab fill in host name/address as `POSTGRES_HOSTNAME`, username as `MDS_USER` and password as `MDS_PASSWORD` as defined in your `.env` file

### Run the ingest

1. Go to the ingest/ directory and run the `cp .config.sample .config` command
    1. Follow the instructions in the .config file to create your config. A sample config looks something like this:
    ```
    [DEFAULT]
        ref = master
    [2411d395-04f2-47c9-ab66-d09e9e3c3251]
        auth_type = x
        token = x
        headers = { "custom": "header" }
    ```
2. Run `docker-compose build --no-cache ingest`  from the root of the repository
    1. **NOTE:** you will have to run this command when you change anything in the `ingest/` directory, including the config or the scripts if you’re debugging
3. Now you’re ready to ingest data. A couple notes on how this works
    1. If you’re pulling data from a provider API, you can specify an `--output` flag to the ingest command to output the data to a directory once the data pull is complete. **Note:** this is supposed to be a directory in the docker container, for instance `/home/jovyan/work/mds/data` as the output directory. You can then use `docker cp $container_id:/home/jovyan/work/mds/data ./` to copy the files to your local machine. For more information on how the directories are configured in the docker container, look at `ingest/Dockerfile`
    2. You can specify a `--source` flag to tell the ingestor to get data from a file rather than pulling data from a provider API
    3. There’s a `--rate_limit` flag if your endpoint uses rate limiting. Configure this flag using an integer to specify how many seconds the ingest will sleep in between API requests.
    4. There’s a `--stage_first` flag that has to be specified. There’s an [open PR](https://github.com/CityofSantaMonica/mds-provider-services/pull/37) to ask for more clarification around the usage of this flag but based on previous usage of this setting it should be set to 3.
    5. Run the ingest:
    ```bash
    docker-compose run ingest \
        --output "/home/jovyan/work/mds/data/" \
        --providers Bird \
        --status_changes \
        --no_validate \
        --start_time INTEGER_EPOCH \
        --end_time INTEGER_EPOCH \
        --stage_first 3 \
        --rate_limit 1
    ```
    6. **Note:** the docs tell you to run the ingest via `docker-compose run --rm ingest [OPTIONS]` but the `--rm` flag will remove the container once this is done running. If you’re specifying an `--output` flag and want the container to persist so you can pull the json files out, then do not use the `--rm` flag.
    7. We use `--no_validate` because we separate validation and ingestion to ensure we are validating against the most recent version of MDS. In the past, we ran with the `--no_validate` flag because it behaved differently than the mds-validator.
    8. It is recommended to read the [ingest function](https://github.com/CityofSantaMonica/mds-provider-services/blob/master/ingest/main.py#L258) to understand the steps of what the ingest script does. Errors in ingest may cause all data pulled to be lost.

### Running the availability calculation

1. Run the steps [here](https://github.com/CityofSantaMonica/mds-provider-services/tree/master/db#availability) to create the views for the availability calculation
2. Then run `docker-compose build --no-cache analytics` and then `bin/notebook.sh analytics` to start the python notebook
