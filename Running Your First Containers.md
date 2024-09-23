## Example 1: Whalesay Container

1. **Run the Whalesay container:** Execute the following command in your terminal or command prompt:
```
docker run docker/whalesay cowsay "Hey Team! 👋"
```
This command will pull the public whalesay image from Docker Hub, download it, and run a container with the custom command provided.

2. **View the output:** After the container finishes running, you should see an ASCII art output with the custom phrase "Hey Team! 👋".

```bash
 ________________
< Hey Team! 👋 >
 ----------------
    \
     \
      \
                    ##        .
              ## ## ##       ==
           ## ## ## ##      ===
       /""""""""""""""""___/ ===
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
       \______ o          __/
        \    \        __/
          \____\______/
``````
## Example 2: Postgres Container

1. **Run the Postgres container:** Execute the following command in your terminal or command prompt:
```bash
docker run -e POSTGRES_PASSWORD=foobarbaz -p 5432:5432 postgres:15.1-alpine
```
This command will:

- Set the `POSTGRES_PASSWORD` environment variable (required for the container to start)
- Publish port `5432` on your localhost and connect it to port `5432` inside the container
- Pull the `postgres:15.1-alpine` image from Docker Hub, if it's not already available locally
- Start the container, creating a running Postgres 15.1 instance on the Alpine operating system
2. **Verify the Postgres container is running:** Open a PostgreSQL client such as pgAdmin and connect to the database by adding a new server with the following information:
```bash
   - Host: localhost
   - Port: 5432
   - User: postgres
   - Password: foobarbaz
```
3. **Execute a query:** Run a sample query to confirm the connection and access to the database inside the container. For example:
```sql
SELECT * FROM information_schema.tables;
```
This query will return all the tables from the information schema within the database running inside the container.