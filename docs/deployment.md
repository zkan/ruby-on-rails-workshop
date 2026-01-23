# Deployment

เราใช้เครื่องมือที่ติดมากับ Rails ที่ชื่อว่า [Kamal - Deploy web apps
anywhere](https://kamal-deploy.org/){target=_blank} ในการ deploy Rails app
ขึ้นไปบน server

Kamal (แบบ minimum ที่ใช้ Docker registry) ต้องใช้ข้อมูลตามนี้

1. Server IP
1. Public key ของคน deploy
1. Docker personal access token สำหรับ pull/push Docker image
1. Registry URL ของ Docker
1. Web domain

ไฟล์ `config/deploy.yml`

```yaml
# Name of your application. Used to uniquely configure containers.
service: kan_todo_app_rails

# Name of the container image (use your-user/app-name on external registries).
image: zkan/todo-app-rails

# Deploy to these servers.
servers:
  web:
    - 35.233.169.175

proxy:
  ssl: true
  host: todo-kan.odds.team

# Where you keep your container images.
registry:
  # Alternatives: hub.docker.com / registry.digitalocean.com / ghcr.io / ...
  server: registry-1.docker.io

  # Needed for authenticated registries.
  username: zkan

  # Always use an access token rather than real password when possible.
  password:
    - KAMAL_REGISTRY_PASSWORD

# Inject ENV variables into containers (secrets come from .kamal/secrets).
env:
  secret:
    - RAILS_MASTER_KEY
  clear:
    # Run the Solid Queue Supervisor inside the web server's Puma process to do jobs.
    # When you start using multiple servers, you should split out job processing to a dedicated machine.
    SOLID_QUEUE_IN_PUMA: true

# Aliases are triggered with "bin/kamal <alias>". You can overwrite arguments on invocation:
# "bin/kamal logs -r job" will tail logs from the first server in the job section.
aliases:
  console: app exec --interactive --reuse "bin/rails console"
  shell: app exec --interactive --reuse "bash"
  logs: app logs -f
  dbc: app exec --interactive --reuse "bin/rails dbconsole --include-password"

# Use a persistent storage volume for sqlite database files and local Active Storage files.
# Recommended to change this to a mounted volume path that is backed up off server.
volumes:
  - "kan_todo_app_rails_storage:/rails/storage"

# Bridge fingerprinted assets, like JS and CSS, between versions to avoid
# hitting 404 on in-flight requests. Combines all files from new and old
# version inside the asset_path.
asset_path: /rails/public/assets

# Configure the image builder.
builder:
  arch: amd64

# Use a different ssh user than root
ssh:
  user: zkan.cs
  keys: ["~/.ssh/id_ed25519"]
```

โดยที่ IP ของ server ในที่นี้คือ `35.233.169.175`

ส่วนค่า `KAMAL_REGISTRY_PASSWORD` คือ Docker personal access token (ในกรณีที่เราใช้
Docker registry) ซึ่งจะดึงค่ามาจากไฟล์ `.kamal/secrets` ที่โค้ดด้านล่างตามนี้ (ถ้าโค้ดนี้ถูก
comment อยู่ ก็ให้ uncomment ก่อน)

```
# Grab the registry password from ENV
KAMAL_REGISTRY_PASSWORD=$KAMAL_REGISTRY_PASSWORD
```

นั่นแปลว่าก่อนที่เราจะรันคำสั่ง `kamal` เราจะต้องมีการเซต environment variable ที่ชื่อ
`KAMAL_REGISTRY_PASSWORD` ก่อน

เมื่อเราเซตทุกอย่างครบแล้ว ให้สั่ง

```bash
kamal setup
```

## References

* [Kamal -
Installation](https://kamal-deploy.org/docs/installation/){target=_blank}
