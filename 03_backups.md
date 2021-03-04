# Data Engineering Toy Project
## Backups

I realized I should backup my sqlite database in case my instance fails (or I break it).

`gcloud auth login` to login to account

`gcloud init` to initialize the gcloud command line tool

`gsutil` is used for google storage (AWS S3 equivalent).

`gsutil ls` lists my buckets.

`gsutil mb gs://<name>` creates a bucket.

`gsutil cp /home/pdywilson/rentcrawler/db/rent.db "gs://rent_db_backup/$(date)/"` copies my db to the gstorage into a folder named after the current date. This is how I can automate backups and have them nice and tidy.


