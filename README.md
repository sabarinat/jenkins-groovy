Start
  |
  v
+------------------------+
| Clean Workspace        |
| rm -rf workspace/*     |
+------------------------+
  |
  v
+------------------------+
| Get Credentials from   |
| JSON (withCredentials)|
| Parse docker_image     |
| Set env variables      |
+------------------------+
  |
  v
+------------------------+
| Git Clone/Pull kube    |
| kube-bas-learning repo |
+------------------------+
  |
  v
+------------------------+
| Docker Login           |
| docker login with env  |
+------------------------+
  |
  v
+------------------------+
| Git Clone/Pull Node.js |
| simple-node-docker repo|
+------------------------+
  |
  v
+------------------------+
| Docker Build & Push    |
| Check if image exists  |
| If not:                |
| - Checkout …
