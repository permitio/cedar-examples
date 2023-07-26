# TinyTodo - OPAL and Cedar Agent fork

TinyTodo is a simple task list management application. It uses OPAL and Cedar Agent to control who has access to what.

TinyTodo allows `Users` to organize, track, and share their to-do lists. `Users` create `Lists`, which they can populate with tasks. As tasks are completed, they can be checked off the list. A list's creator, called _owner_, can share a list with `Users` and `Teams`, assigning them with either _reader_ or _editor_ roles. A reader can only view the contents of a list, while an editor can modify it (e.g., add tasks, or check them off the list).

## What is OPAL?

[OPAL stands for_Open Policy Administration Layer_](https://github.com/permitio/opal). It has two main components: a server and a client. The server tracks the state of the policy on Git or bundles, the Client is responsible for saving the Policy and the Data so that the policy engine can evaluate the policy.
OPAL supports multiple engines for policy evaluation, such as OPA and Cedar, with additional engine support upcoming.
[You can read more about OPAL here](https://opal.ac).
You can support the project by giving it [a star on GitHub](https://github.com/permitio/opal).

## What is Cedar Agent?

[Cedar Agent](https://github.com/permitio/cedar-agent) is an HTTP server designed to manage the policy and data stores efficiently.
It provides a seamless integration with [Cedar](https://www.cedarpolicy.com/en), a language for defining permissions as
policies.  
With Cedar-Agent, you can easily control and monitor access to your application's resources by leveraging Cedar
policies.
You can support the project by giving it [a star on GitHub](https://github.com/permitio/cedar-agent).

## Permit.io

[Permit.io](https://permit.io) is a platform for managing access to your application's resources. It builds on top of OPAL and Cedar Agent to provide a complete solution for managing access to your application's resources. Permit also provides a UI for managing the policies and the data.

## Usage

The code is structured as a server that processes HTTP commands and is written in Rust. The client, `tinytodo.py`, written in Python3, can be used to interact with the server. As this is just a demo app, so there is no permanent storage of to-do lists -- they last only as long as the server is running.

### Build

You need Docker Python3 and Rust. Rust can be installed via (rustup)[https://rustup.rs]. Python3 can be installed (here)[https://www.python.org/] or using your system's package manager. Docker can be installed (here)[https://docs.docker.com/get-docker/].

Install the required python packages, and build the server as follows:
```shell
pip3 install -r requirements.txt
cargo build --release
```
The Rust executable is stored in `target/release/tiny-todo-server`.

### Run OPAL server and Python client
To start opal-server and opal-client, run
```shell
docker compose -f docker-compose-example-cedar.yml up
```

To start the client within Python interactive mode, enter:
```shell
python3 -i ./tinytodo.py // it will start the rust server automatically and set the user as andrew (admin)
```

The OPAL server reads Cedar policies from a pre-defined GitHub account. The policies are then passed to the OPAL client, and from there to the Cedar Agent. The Cedar entities which define the TinyTodo `Users` and `Teams` from `data/data.json` are sent to the Cedar Agent by OPAL as well.

Look at the `tinytodo.py` code to see the functions you can call, which serve as the list of commands.
```python
set_user("andrew") # set the user as andrew (admin)
get_lists() # get all lists
get_list(list_id) # get list1
create_list("list1") # create list1
create_task(list_id, "task1") # create task1 in list1
toggle_task(list_id, task_id) # toggle task1 in list1
share_list(list_id, "username") # share list1 with user "username"
and more...
```

See also [`TUTORIAL.md`](./TUTORIAL.md) for a detailed description of how to use these commands and how TinyTodo works.

## Main changes from the original TinyTodo

- Entities are synced every time data is changed, thus always keeping them up to date. This is done by calling `save_entities_and_sync()` in `context.rs` after every change (like a new list being created or shared).
- We added a new `get_entities` route to get the entities from the app state so that OPAL can sync them.
- When we check for access, we only send the `user`, `action`, `resource`, and `context` to the Cedar Agent. We don't need to send the policy and the entities every time as OPAL and Cedar Agent already sync them.
```rust
// in context.rs
let client = reqwest::blocking::Client::new();
let res = client.post("http://localhost:8180/v1/is_authorized")
    .json(&serde_json::json!({
        "principal": principal.as_ref().clone().to_string(),
        "action": action.as_ref().clone().to_string(),
        "resource": resource.as_ref().clone().to_string(),
        "context": {}
    }))
    .send();
```
