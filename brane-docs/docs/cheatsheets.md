**Brane -- Cheat sheet**

**(Last update -- 14-10-2025)**

**Links BRANE**

- [Brane GitHub Issues](https://github.com/BraneFramework/brane/issues)

- [Brane Documentation](https://braneframework.github.io/) 
-  <https://braneframework.github.io>

**Release and Maintenance**

-  **Last stable release (v3.0.0)** -- *no longer maintained.*

-  Development is active only through **nightly builds**, which produce
  a **Brane distribution** containing the latest experimental
  features.

**Note:**

This cheat-sheet document summarizes the important steps to perform as
end-user, software engineer, or system admin to *install Brane* and to
*deploy* Brane on *one node* (local tests) and on *multiple nodes*. The
cheat sheets also cover the process of creating *Brane functions*, and
*BraneScript* to compose data prepressing pipeline (workflow). The cheat
sheets to do no give a detailed explanation of the Brane concepts for a
deep understanding it is recommended to read the full manual

## **Brane --- tools: brane, branectl**
-  **brane** -- User CLI used by scientists/software engineers
  to **submit and run workflows** to Brane
-  branectl --CLI used to **install, configure, and manage
  nodes** (control, worker, proxy) in a Brane.

#### Brane --- branectl 

```bash
Usage: branectl [OPTIONS] <COMMAND>
```
###### Commands and Options

| **Command / Option** | **Description** |
|------------------------|-----------------|
| `services <type>` | Downloads prebuilt Docker images for Brane services (`central`, `worker`, or `proxy`). |
| `generate infra` | Creates the `infra.yml` configuration listing worker nodes. |
| `generate proxy` | Generates the `proxy.yml` configuration for proxy setup. |
| `generate backend` | Creates the `backend.yml` file defining the compute backend. |
| `generate node` | Generates the main `node.yml` configuration for any node. |
| `generate certs` | Creates certificates (`ca.pem`, `server.pem`, `client-id.pem`) for authentication. |
| `generate policy_secret` | Generates private key files for the Brane policy service. |
| `generate policy_db` | Initializes the local policy database (`policies.db`). |
| `generate policy_token` | Creates access tokens for policy experts to manage policies. |
| `start` | Starts a Brane node (`central`, `worker`, or `proxy`) using Docker. |
| `--debug` | Enable debug mode. |
| `--skip-check` | Skip dependency checks. |
| `-h, --help` | Print help information. |
| `-v, --version` | Print version information. |

#### Brane --- brane

The Brane command-line interface

```bash
[Usage:] brane [OPTIONS] <COMMAND>
```

###### Commands and Options

| **Command / Option** | **Description** |
|------------------------|-----------------|
| `certs` | Manage certificates for connecting to remote instances. |
| `data` | Data-related commands. |
| `instance` | Commands related to connecting to remote instances. |
| `package` | Commands related to packages. |
| `upgrade` | Upgrades outdated configuration files in the current Brane version. |
| `verify` | Verifies parts of Brane's configuration. |
| `version` | Shows the version number for this Brane CLI tool and (if logged in) the remote Driver. |
| `workflow` | Commands related to workflows. |
| `help` | Prints this message or the help of a specific subcommand. |
| `--debug` | Enable debug mode. |
| `--skip-check` | Skip dependency checks. |
| `-h, --help` | Print help information. |
| `-V, --version` | Print version information. |

## **Install --- Brane on local host**   

Commands for a--- to download branectl locally:

#### 1. **Download binary:** 

Downloads the precompiled branectl binary for

Intel Macs
```bash
sudo wget -O /usr/local/bin/branectl 
  https://github.com/braneframework/brane/releases/latest/download/branectl-darwin-x86_64
```

#### 2. **Make it executable:** 

Grants permission to run the binary

```bash
sudo chmod +x /usr/local/bin/branectl
```

#### 3. **Verify installation:** 

Confirms that branectl is installed and working

```bash
branectl --help
```

## **Installation --- multi-nodes** 

the **manual describes how to set up multi-node Brane**:

-  A **control node** (central orchestrator).

-  Multiple **worker nodes** (compute sites).

-  Optional **proxy nodes** (network intermediaries).

The manual gives detailed steps using branectl to generate configs
(**infra.yml**, **node.yml**), add certificates, and start each node so
they work together in one Brane instance.

#### 1. **Prepare machines & install dependencies** 

on each host composing the cluster:

-  install Docker, Docker BuildKit,

-  OpenSSL

-  ensure GLIBC ≥ 2.27.

#### 2. **Install branectl** 

-  download the precompiled branectl binary for each node
    (**control, worker, proxy)**

-  make it executable: chmod +x branectl.

#### 3. **Create certificates**

on the machine that will start nodes, run 

```bash 
branectl generate certs 
```

> to create certificates (**ca.pem**, **server.pem**, **client-id.pem**)
> for node authentication.

#### 4. **Obtain Brane services images** 

On each node (**control, worker, proxy)**

```bash
branectl download services central| worker|proxy 
```

> to download prebuilt Docker images for Brane services (central, worker, or proxy).

#### 5. **Generate central [infra file]** --- run 

```bash
branectl generate infra <ID>:<ADDR> ... 
```

To create **config/infra.yml** listing worker IDs and addresses.

-  **ID** → unique name of each worker (e.g., amy, bob)
-  **ADDR** → IP address or hostname (e.g., 192.0.2.2)

> More details about how to write configuration files see [[Configuration
files manual](https://braneframework.github.io/manual/config/admins/infra.html)]

#### 6. **Generate central [proxy config]** --- run 

```bash
branectl generate proxy -f -p ./config/proxy.yml 
```
> to create **proxy.yml** (use defaults unless advanced networking).

> More details about how to write configuration files see [[Configuration files
manual](https://braneframework.github.io/manual/config/admins/proxy.html)]

#### 7. **Generate central [node file]** --- run

```bash
branectl generate node -f central <HOSTNAME> 
```
> to produce **node.yml** pointing to infra and proxy files.

> More details about how to write configuration files see
> [[Configuration files manual](https://braneframework.github.io/manual/config/users/data.html)]

#### 8. **Generate [worker backend config]** ---run (on each worker node)

```bash
branectl generate backend -f -p ./config/backend.yml local 
```
(or appropriate backend).

> More details about how to write configuration files see [[Configuration files manual](https://braneframework.github.io/manual/config/admins/backend.html)]

#### 9. **Generate [worker proxy]**--- run on each worker.

```bash
branectl generate proxy -f -p ./config/proxy.yml 
```

#### 10. **Generate [worker node file]}** 

to create the worker's **node.yml**

```bash
branectl generate node -f worker <CENTRAL_HOSTNAME> <LOCATION_ID> .
```
> <CENTRAL_HOSTNAME>
> <LOCATION_ID> 

#### 11. **Generate certificates (server & client)** 

On each worker/proxy run 

```bash
branectl generate certs -f -p ./config/certs server <LOCATION_ID> -H <HOSTNAME> 
```

and create client certs as needed with 

```bash
branectl generate certs client ....
```

#### 12. **Add [worker certs] to the [central node**]--- create 
 

> **Share certificates** --- copy each worker's **ca.pem** (and client
> certs if required) to peers under **config/certs/peer/** so mutual
> auth works.

#### 13. **Generate policy tokens & [policy secrets (optional)**  --- run 

**[policy token]**

```bash
branectl generate policy_token <INITIATOR> <SYSTEM> <DURATION> -s <PATH_TO_SECRE>
```
and give **policy_token.json** to the policy expert.

**[policy secrets]** 

```bash
branectl generate policy_secret -f -p ./config/policy_*.json 
```

> More details about how to write data policy see [[Configuration files manual](https://braneframework.github.io/manual/policy-experts/introduction.html)]

#### 14. **Start services** --- on central: 

```bash
branectl start central
```

on each worker: 

```bash
branectl start worker
```

on proxies: 
```bash
branectl start proxy.
```

#### 15. **Wait for DB readiness & verify** 

--- wait until Scylla is fully up

verify with docker ps that all Brane containers (including brane-api) are running.

## **Create Brane Function**  

The end user must use **brane** (the user CLI) to package and publish
code as a Brane function. The general sequence of steps to create a
brane function is:

#### 1. **Initialize a new package:** 

Creates a new Brane package directory

```bash
brane new
```

#### 2. **Add your Python script:**

> Place helloworld.py inside the package folder

#### 3. **Define the function interface:**

> Edit the package definition file
> (e.g., **container.yml** or **data.yml**) to describe the function
> (inputs, outputs, command).

The following configuration files are relevant for users of the
framework:

-  [container.yml](https://braneframework.github.io/manual/config/users/container.html):
  A **YAML** file that defines the metadata of a BRANE package.

-  [data.yml](https://braneframework.github.io/manual/config/users/data.html):
  A **YAML** file that defines the metadata of a BRANE dataset.

> More details can be found in Brane manual [[your first package](https://braneframework.github.io/manual/software-engineers/hello-world.html),
package input-multiple functions, datasets, ]

#### 4. **Build the package:** 

Builds the package into a Brane container image

```bash
brane build
```
#### 5. **Push the package to a registry:** 

Publishes the package so it can be used in workflows

```bash
brane push
```

#### 6. **Run or test it:** 

Executes the function in the Brane environment

```bash
brane run helloworld
```
***Note**:* 

## **Run Workflow - BraneScript**  

BraneScript (the scripting language for composing workflows), DSL for
**scientists** to orchestrate workflows.

#### 1. **Write a BraneScript file manually** --


 Create a text file, e.g. **workflow.bs**, that defines the workflow
 using BraneScript syntax (calls to packages/functions)*.*

 example HelloWorld.bs
 

```
// HELLO WORLD.bs
// by Rick Sanchez
//
// A BraneScript workflow for printing "Hello, world!" to the screen.
// Define which packages we use, which makes its functions available
// ('hello_world()', in this case)

import hello_world;

// Prints the result of the 'hello_world()' call by using the 'println()' builtin
println(hello_world());
```

#### 2. **Run the workflow** 

Execute the BraneScript workflow through the Brane CLI

**Local execution**: test your workflow locally first to make sure
 that it works and compiles fine without consuming instance resources

 Test the function packages available

```bash
 brane package list
```

if the package you want to use is not listed, download it form Brane
GitHub1

 brane package import braneframework/brane-std
 hello_world/container.yml

 We assume you're already executed 'brane instance add' ????

```bash
 brane workflow run HelloWorld.bs
```

 **Remote execution:**

```bash
brane workflow run --remote HelloWorld.bs
```

> More details about how to write BraneScript see [[BraneScript manual](https://braneframework.github.io/manual/branescript/introduction.html)]

## **Run Workflow - Read-Eval-Print Loop (REPL)**  

An alternative to writing BranScripts in files is the Brane
Read-Eval-Print Loop (REPL).

Start the REPL

```bash
 brane workflow repl
```

reproduce our Hello Word workflow by writing its two statements
separately:

```python
// In the Brane REPL
import hello_world;
println(hello_world());
```

use the REPL in a remote setting

```bash
brane workflow repl --remote
```

### **Create Data Policy in eFLINT**  

#### TODO  

### **Data Policy Graphical user Interface**  

#### TODO   More details can be found in Brane manual [ [Policy Reasoner GUI](https://github.com/braneframework/policy-reasoner-gui)]

### **JupyterLab Graphical user Interface**

#### TODO  

