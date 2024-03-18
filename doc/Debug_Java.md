
# Debug Java component

It is disgusting to use the commandline tool `jdb` to debug Java. Let alone debugging Java component of a big software such as LibreOffice. We need a better, morden, and faster way.

A better tool is required, but there is no decent and stable IDE on riscv64 till now. Besides, the IDE on x86_64 is mature. It might be a good choice to use x86 IDE to debug Java program on riscv64.

## 1. Set SSH tunnel

```shell
# execute this command on client side.
ssh -N -L 14286:localhost:24286 debian@101.230.251.34 -p 38002
```

Our goal is to let ssh-client(on x86) listen to **one local port**, and then send local data though **the open port** on the server side. After the ssh-server received data from **the open port**, it forward the data to **one local port** on the server.

In the above command,

- `14286` is the local port that ssh-client listens to.
- `24286` is the local port on the server side that the ssh-server forward the data to.
- `38002` is the open port of the ssh-server.
- `-L` means creating the ssh tunnel with local port forward.

## 2. Configure x86 IDEA

Our goal in this step is to let IDEA forward the data to the local port to get the data processed by ssh-client.

First we need a copy of the source code on the riscv64 server. Open the source code folder with IDEA, excute `Run`->`Edit configurations`, and add `Remote JVM Debug` -- set `host` to `localhost`, `port` to `14286`.

## 3. Add riscv64 LibreOffice Java library

Our goal in this step is to fix the symbol missing error in IDEA.

Clone all the `.jar` files in `instdir/program` to the x86 machine. Execute `File`->`Project Structure`->`Project Settings`->`Modules`->`Dependencies`, and add the folder path containing the `.jar` files.

## 4. Run Java program on riscv64 server

Our goal in this step is running the program on riscv64 server, and expose debug port to the corresponding port on server side.

Add this segment to the Java command you want to run.

```shell
-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=*:24286
```

if `suspend=y`, the program will suspend until a debugger attached.

## 5. Remote debug on x86 IDEA

In IDEA, execute `Run`->`Debug...`, and choose the Remote JVM Debug configuration above.

Enjoy it!
