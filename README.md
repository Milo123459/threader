# threader
Intensive tasks made simple

## What

Run intense tasks in a background thread. This allows you to then continue with your workflow.


You can then use `threader view` to view the command running, it's output and the commands in [queue](#threader-queue). If you run `threader ls` it will list the instances of Threader running, which is probably going to be 1. If you want to spawn more then 1, type `threader new`.

### Advanced workflow

```sh
threader ls # list instances
1: not active

threader new # create a new instance
2: not active

threader ls # list instances
1: not active
2: not active

threader run "cargo install cargo-audit" # install cargo audit in instance 1 of threader
1: active

threader run "cargo audit" # wait for the previous task in threader instance 1 to finish executing
queued

threader run -i 2 "echo i execute after cargo audit is done!" --wait 1 # waits for all commands queued / running in threader instance 1 execute before spawning the command
2: active

threader run -i 2 "echo destroying... :(" -d # waits for the previous command in instance 2 to be done, then execute this, persist logs and destroy instance
queued
```

Say you have 2 threader instances running, and you want to run a command after a specific command in instance 1 is done executing, not after all the commands in queue are done executing. This can be achieved via this command:
```sh
threader run -i 2 --wait 1 --wait-for-command "cargo install cargo-audit" "cargo audit"
```

`--wait-for-command` takes the command that will be run and then waits for that to be executed.

#### Real life use cases

```sh
threader new
2: not active

threader run "cargo install cargo-audit"
1: active

theader run "cd my_cool_project"
queued

threader run "cargo audit"
queued

threader run -i 2 "cd my_other_project"
2: active

threader run --wait 1 --wait-for-command "cargo install cargo-audit" -i 2 "cargo audit"
queued
```

This installs cargo-audit on instance 1, whilst that is happening instance 2 has CDed into a directory. Once the installation has finished, threader instance 1 cds into a directory whilst instance 2 runs `cargo audit`. Once instance 1 has finished CDing, it'll then run `cargo audit`.

### Threader queue

A queue system, that waits for the previous command to finish executing before running the next one.