# minecraft-docker

                                      ┌─────────────────────┐
                                      │     GitHub Repo      │
                                      │  - server configs    │
                                      │  - plugins/mods      │
                                      │  - backup scripts    │
                                      └─────────┬───────────┘
                                                │ Pull / Update
                                                ▼
                                      ┌─────────────────────┐
                                      │   Host Machine / PC  │
                                      │  - Minecraft Server │
                                      │    (Java/Bedrock)   │
                                      │  - Docker optional  │
                                      │  - Local scripts    │
                                      └─────────┬───────────┘
                                                │ Reads / Writes
                             ┌──────────────────┼──────────────────┐
                             │                  │                  │
                             ▼                  ▼                  ▼
                   ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
                   │  World Folder   │  │ Local Database  │  │ Backup Scripts  │
                   │  (/mc-world)    │  │ PostgreSQL /    │  │ Powershell,     │
                   │  Persistent     │  │ MySQL locally   │  │ Cron jobs       │
                   └─────────────────┘  └─────────────────┘  └─────────────────┘


Explanation
GitHub Repo
Stores all your configs, plugins, mods, and scripts.
You can pull updates whenever you want.
Host Machine / PC
Runs the Minecraft server.
Can run directly or inside Docker for isolation.
Docker can mount your local world folder so everything persists outside the container.
World Folder
Stored locally on your PC.
Docker or direct server reads/writes here.
Local Database
All plugins or mods that need a DB can connect here.
Stays local for performance.
Backup Scripts
Can backup the world folder, configs, and database.
Run automatically or manually.

This way you get versioned configs, a persistent world, local database, and optional Docker isolation.






---

Perfect! Let’s set up a local Minecraft server with Docker that:

Uses your PC for CPU/RAM/network
Stores worlds, plugins, and configs in local folders
Can connect to a local Postgres database if needed
Can be updated via GitHub (for configs or mods)

Here’s a clean example:

1. Create a folder structure on your PC
mkdir -p ~/minecraft-server/world
mkdir -p ~/minecraft-server/plugins
mkdir -p ~/minecraft-server/config
mkdir -p ~/minecraft-server/data
world → your Minecraft world files
plugins → server plugins or mods
config → server configuration files (server.properties, etc.)
data → any extra data like scripts, backups
3. Create a Dockerfile

In ~/minecraft-server/Dockerfile:

# Use a lightweight Java image
FROM openjdk:17-jdk-slim

# Create app directory inside container
WORKDIR /minecraft

# Copy server jar
COPY server.jar .

# Expose default Minecraft port
EXPOSE 25565

# Mount points for external volumes
VOLUME ["/minecraft/world", "/minecraft/plugins", "/minecraft/config", "/minecraft/data"]

# Run the server
CMD ["java", "-Xmx2G", "-Xms1G", "-jar", "server.jar", "nogui"]

Notes:

server.jar should be your downloaded Minecraft server jar (Vanilla, PaperMC, etc.)
Adjust -Xmx2G to the RAM you want the server to use
3. Create a docker-compose.yml (optional, cleaner)
version: '3.8'

services:
  minecraft:
    build: .
    container_name: mc-server
    ports:
      - "25565:25565"
    volumes:
      - ./world:/minecraft/world
      - ./plugins:/minecraft/plugins
      - ./config:/minecraft/config
      - ./data:/minecraft/data
    restart: unless-stopped

This allows you to just run:

docker compose up -d
4. Add GitHub integration for configs/plugins
Clone your repo into ~/minecraft-server/config or ~/minecraft-server/plugins
Run a script to git pull before starting the server, either manually or with a small wrapper script.
Docker will always read your local folders, so updated files in Git are reflected immediately.
5. Optional: connect to Postgres

If your plugins or mods need a database:

    environment:
      - DB_HOST=host.docker.internal
      - DB_PORT=5432
      - DB_USER=youruser
      - DB_PASS=yourpass

host.docker.internal points to your host machine from inside Docker, so your container can access your local Postgres database.

✅ Advantages of Docker here:

Easy reset: if something breaks, just remove container and start fresh
Isolation: server won’t mess with your system Java or files
Port mapping: you control exactly which ports are open
Versioning: multiple server versions can run without conflicts
