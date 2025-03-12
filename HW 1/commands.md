## network
docker network create pg-network
# create a local volume
docker volume create --name dtc_postgress_volume_local -d local
# running postgress database
docker run -it \
  -e POSTGRES_USER="admin" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_green_taxi" \
  -v dtc_postgress_volume_local:/var/lib/postgresql/data \
  -p 5432:5432 \
  --network pg-network \
  --name pg-database \
  postgres:13
# run pg-admin
docker run -it\
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com"\
  -e PGADMIN_DEFAULT_PASSWORD="root"\
  -p 8080:80\
  --network pg-network\
  --name pg-admin\
  dpage/pgadmin4
# for loading data into the database via python script
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz"

python ingest-data.py \
  --user=root \
  --password=root \
  --host=localhost \
  --port=5432 \
  --db=ny_taxi \
  --tb=yellow_taxi_trips \
  --url=${URL}

# building the docker image
docker build -t green_taxi_ingest:v001 .
# running the docker image
docker run -it \
  --network=pg-network \
  green_taxi_ingest:v001 \
    --user=admin \
    --password=root \
    --host=pg-database \
    --port=5432 \
    --db=ny_green_taxi \
    --table_name=green_taxi_trips \
    --url=${URL}