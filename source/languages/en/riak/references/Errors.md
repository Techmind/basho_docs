---
title: Common Errors List
project: riak
version: 1.3.0+
document: reference
toc: true
audience: advanced
keywords: [errors]
---

This is not a comprehensive listing of every error that Riak can encounter -- screws fall out all of the time, the world is an imperfect place. This is an attempt at capturing the most common recent errors that users do encounter, as well as give some description to non critical error atoms which may be eoncountered in logs.

### Message Format

The tables below do not specify which logs these error messages may appear in. Depending upon your log configuration some may appear more often -- if you're set to log debug, while others may output to your console (eg. if you tee'd your output or started as `riak console`). This is organized to be able to lookup a portion of a log message.

Riak does not format all error message that it receives into human-readable sentences. However, It does output many errors as objects.

22:35:17.862 [error] gen_server riak_core_capability terminated with reason: no function clause matching orddict:fetch('riak@192.168.2.101', []) line 72

## erlang

Although relatively rare once a Riak cluster is running in production, users new to Riak or Erlang occasionally encounter errors on initial installation. These spring from a setup Erlang does not expect, generally due to network, permission, or configuration problems.

Error    | Message | Description | Resolution
---------|---------|-------------|-------
{error,duplicate_name} | | You are trying to start a new Erlang node, but another node with the same name is already running | You might be attempting to start multiple nodes on the same machine with the same `vm.args` `-name` value. Or Riak is already running, check for beam.smp. Or epmd thinks Riak is running, check/kill epmd.
{error,econnrefused} | | Remote erlang node connection refused | Could be a few problems. Ensure your remote `vm.args` `-setcookie` must be the same value for every node in the cluster. The `vm.args` `-name` value must not change after joining the node (unless you use `riak-admin cluster replace`). Ensure your machine does not have a firewall or other issue that prevents traffic to the remote node.
{error,enoent} | | Missing an expected file or directory | Ensure all `*_dir` values in `app.config` exist, for example, `ring_state_dir`, `platform_data_dir`, and others.
system_memory_high_watermark | | Often a sign than an Erlang ets table has grown too large | Check that you are using a backend appropriate for your needs (leveldb for very large key counts), that your vnode count is reasonable (measured in dozens per node rather than hundreds).

## riak_kv

Many KV errors have prescriptive messages. For example, the mapreduce parse_input phase will respond thusly with an invalid input:

"Inputs must be a binary bucket, a tuple of bucket and key-filters, a list of target tuples, or a search, index, or modfun tuple: INPUT".

For such cases we leave it to Riak to explain the correct course of action. For the remaining common error codes, they are often marked by Erlang atoms. This document describes those terse error codes.

Crash on Startup
CRASH REPORT
INFO REPORT

Error    | Message | Description | Resolution
---------|---------|-------------|-------
all_nodes_down |  | No nodes are available | Check `riak-admin member-status` and ensure that all expected nodes in the cluster are of `valid` Status
{bad_qterm, QueryTerm} |  | Bad query when performing mapreduce | Fix your mapreduce query
{coord_handoff_failed, Reason} | Unable to forward put for `Key` to `CoordNode` - `Reason` | Vnodes unable to communicate | Ensure your cluster is up and nodes are able to communicate with each other. Check that coordinating vnode is not down.
{could_not_reach_node, Node} |  | Erlang process was not reachable | Check network settings; ensure remote nodes are running and reachable; ensure all nodes have the same erlang cookie setting `vm.args` `-setcookie`
{deleted, Vclock} |  | The value was already deleted, includes the current vector clock | Riak will eventually clean up this tombstone
{dw_val_violation, DW} |  | Same as `w_val_violation` but concerning durable writes | Set a valid DW value
{field_parsing_failed, {Field, Value}} | Could not parse field `Field`, value `Value`. | Could not parse an index field | Most commonly an _int field which cannot be parsed. For example a query like this is invalid /buckets/X/index/Y_int/BADVAL, since BADVAL should instead be an integer
{hook_crashed, {Mod, Fun, Class, Exception}} | Problem invoking pre-commit hook | Precommit process exited due to some failure | Fix the precommit function code, follow the message's exception and stacktract to help debug
{indexes_not_supported, Mod} |  | The chosen backend does not support indexes (only eleveldb currently support 2i) | Set app.config to use the eLevelDB backend
{insufficient_vnodes, NumVnodes, need, R} |  | R was set greater than the total VNodes | Set a proper R value. Or too many nodes are down. Or too many nodes are unavailable due to crash or network partition. Ensure all nodes are available by running riak-admin ring-status.
{invalid_hook_def, HookDef} | Invalid post-commit hook definition `Def` | No Erlang mod and funor javascript functionname | Define the hook with the correct settings
{invalid_inputdef, InputDef} |  | Bad inputs definitions when running mapreduce | Fix inputs settings; Set mapred_systemfrom legacy to pipe
{invalid_range, Args} |  | Index range query hasStart > End | Fix your query
{invalid_return, {Mod, Fun, Result}} | Problem invoking pre-commit hook `Mod`:`Fun`, invalid return `Result` | The given precommit function gave an invalid return for the given `Result` | Ensure your pre-commit functions return a valid result
invalid_storage_backend | storage_backend `Backend` is non-loadable. | Invalid backend choice when starting up Riak | Set a valid backend in app.config (eg.{storage_backend, riak_kv_bitcask_backend})
key_too_large |  | The key was larger than 65536 bytes | Use a smaller key
local_put_failed |  | A local vnode PUT operation failed | Try put again
{n_val_violation, N} |  | (W > N) or (DW > N) or (PW > N) or (R > N) or (PR > N) | No W or R values may be greater than N
{nodes_not_synchronized, Members} |  | Rings of all members are not synchronized | Backups will fail if nodes are not synchronized
{not_supported, mapred_index, FlowPid} |  | Index lookups for mapreduce are only supported with Pipe | Set mapred_system from legacy to pipe
notfound |  | No value found | Value was deleted, or was not yet stored or replicated
{pr_val_unsatisfied, PR, Primaries} |  | Same as `r_val_unsatisfied` but only counts `Primary` node replies | Too many primary nodes are down or the `PR` value was set too high
{pr_val_violation, R} |  | Same as `r_val_violation` but concerning `Primary` reads | Set a valid `PR` value
precommit_fail | Pre-commit hook `Mod`:`Fun` failed with reason `Reason` | The given precommit function failed for the given `Reason` | Fix the precommit function code
{pw_val_unsatisfied, PR, Primaries} | | Same as `w_val_unsatisfied` but only counts `Primary` node replies | Too many primary nodes are down or the `PW` value was set too high
{pw_val_violation, PW} |  | Same as `w_val_violation` but concerning Primary writes | Set a valid `PW` value
{r_val_unsatisfied, R, Replies} |  | Not enough nodes replied to satisfy the `R` value, contains the given `R` value and the actual number of `Replies` | Too many nodes are down or the R value was set too high
{r_val_violation, R} |  | The given R value was non-numeric and not a valid setting (on, all, quorom) | Set a valid R value
receiver_down |  | Remote process failed to acknowledge request | Can occur when listkeys is called
{rw_val_violation, RW} |  | The given `RW` property was non-numeric and not a valid setting (`one`, `all`, `quorom`) | Set a valid `RW` value
{siblings_not_allowed, Object} | Siblings not allowed: `Object` | The hook to index cannot abide siblings | Set the buckets `allow_mult` property to `false`
timeout |  | The given action took too long to reply | Check network settings (ensure no firewall set, iptables, check netstat to see if any of the TCP sockets linger). Or ensure remote nodes are running and reachable. Or ensure all nodes have the same erlang cookie setting `vm.args` `-setcookie`. Or Ensure you have a reasonable `ulimit` size. Note the listkeys can easily timeout and shouldn't be used in production.
{too_few_arguments, Args} |  | Index query requires at least one argument | Fix your query format
{too_many_arguments, Args} |  | Index query is malformed with more than 1 (exact) or 2 (range) values | Fix your query format
too_many_fails |  | Too many write failures to satisfy W or DW | Try write again. Or ensure your nodes/network are healthy. Or set a lower W or DW value.
{unknown_field_type, Field} | Unknown field type for field: `Field`. | Unknown index field extention (begins with underscore) | The only value field types are _int and_bin
{w_val_unsatisfied, RepliesW, RepliesDW, W, DW} | | Not enough nodes replied to satisfy the W value, contains the given W value and the actual number of `Replies*` for either `W` or `DW` | Too many nodes are down or the `W` or `DW` value was set too high
{w_val_violation, W} |  | The given W property was non-numeric and not a valid setting (on, all, quorom) | Set a valid W value
 | Invalid equality query `SKey` | Equality query is required and must be binary for an index call | Pass in an equality value when performing a 2i equality query
 | Invalid range query: `Min` -> `Max` | Both range query values are required and must be binary an index call | Pass in both range values when performing a 2i equality query
 | Failed to start `Mod` `Reason`:`Reason` | Riak KV failed to start for given `Reason` | Several possible reasons for failure, read the attached reason for insight into resolution
 
### Lager

Error | Message
------|--------
 | gen_server `Mod` terminated with reason: `Reason`
 | gen_fsm `Mod` in state `State` terminated with reason: `Reason`
 | gen_event `ID` installed in `Mod` terminated with reason: `Reason`
'function not exported' | call to undefined function `Func` from `Mod` |
undef | call to undefined function `Mod`
bad_return | bad return value `Value` from `Mod`
bad_return_value | bad return value: `Val` in `Mod`
badrecord | bad record `Record` in `Mod`
case_clause | no case clause matching `Val` in `Mod`
function_clause | no function clause matching `Mod`
if_clause | no true branch found while evaluating if expression in `Mod`
try_clause | no try clause matching `Val` in `Mod`
badarith | bad arithmetic expression in `Mod`
badmatch | no match of right hand value `Val` in `Mod`
emfile | maximum number of file descriptors exhausted, check ulimit -n
{system_limit, {erlang, open_port}} | maximum number of ports exceeded
{system_limit, {erlang, spawn}} | maximum number of processes exceeded
{system_limit, {erlang, spawn_opt}} | maximum number of processes exceeded
{system_limit, {erlang, list_to_atom}} | tried to create an atom larger than 255, or maximum atom count exceeded
{system_limit, {ets, new}} | maximum number of ETS tables exceeded
badarg | bad argument in call to `Mod1` in `Mod2`
badarity | fun called with wrong arity of `Ar1` instead of `Ar2` in `Mod`
noproc | no such process or port in call to `Mod`

### Backend Errors

These errors tend to be native, disk, configuration or other server based problems. A network issue is unlikely to affect a backend.

Error    | Message | Description | Resolution
---------|---------|-------------|-------
data_root_not_set | | Same as `data_root_unset` | Set the `data_root` directory in config
data_root_unset | *Failed to create bitcask dir: data_root is not set* | The `data_root` config setting is required | Set `data_root` as the base directory where to store bitcask data, under the `bitcask` section
{invalid_config_setting, multi_backend, list_expected} | | Multi backend configuration requires a list | Wrap `multi_backend` config value in a list
{invalid_config_setting, multi_backend, list_is_empty} | | Multi backend configuration requires a value | Configure at least one backend under `multi_backend` in `app.config`
{invalid_config_setting, multi_backend_default, backend_not_found} | | | Must choose a valid backend type to configure
multi_backend_config_unset | | No configuration for multi backend | Configure at least one backend under `multi_backend` in `app.config`
not_loaded | | Native driver not loading | Ensure your native drivers exist (.dll or .so files {riak_kv_multi_backend, undefined_backend, BackendName} | | Backend defined for a bucket is invalid | Define a valid backed before using this bucket under lib/`project`/priv, where `project` is most likely eleveldb).
reset_disabled | | Attempted to reset a Memory backend in production | Don't use this in production

### Javascript

riak_kv_js_manager, riak_kv_js_vm

Error    | Message | Description | Resolution
---------|---------|-------------|-------
no_vms | *JS call failed: All VMs are busy.* | All Javascript VMs are in use | Wait and run again; Increase Javascript VMs in `app.config` (`map_js_vm_count`, `reduce_js_vm_count`, or `hook_js_vm_count`)
bad_utf8_character_code | *Error JSON encoding arguments: `Args`* | A UTF-8 character give was a bad format | Only use correct UTF-8 characters for Javascript code and arguments
bad_json | | Bad JSON formatting | Only use correctly formatted JSON for Javascript command arguments
 | *Invalid bucket properties: `Details`* | Listing bucket properties will fail if invalid | Fix bucket properties
{load_error, "Failed to load spidermonkey_drv.so"} | The Javascript driver is corrupted or missing | In OSX you may have compiled with `llvm-gcc` rather than `gcc`.

### Mapreduce

Error    | Message | Description | Resolution
---------|---------|-------------|-------
{unknown_content_type, ContentType} | | Bad content type for mapreduce query | Only application/json and application/x-erlang-binary are accepted
 | *Phase `Fitting`: `Reason`* | A general error when something happens using the Pipe mapreduce implementation with a bad argument or configuration | Can happen with a bad map or reduce implementation, most recent known gotcha is when a Javascript function improperly deals with tombstoned objects
bad_mapred_inputs | | A bad value sent to mapreduce | When using the Erlang client interface, ensure all mapreduce and search queries are correctly binary
javascript_reduce_timeout | | Javascript reduce function taking too long | For large numbers of objects, your JavaScript functions may become bottlenecks. Decrease the quantity of values being passed to and returned from the reduce functions, or rewrite as Erlang functions
 | *riak_kv_w_reduce requires a function as argument, not a `Type`* | Reduce requires a function object, not any other type | This shouldn't happen
{'query', Reason} | *An error occurred parsing the "query" field.* | Mapreduce request has invalid query field | Fix mapreduce query
{inputs, Reason} | *An error occurred parsing the "inputs" field.* | Mapreduce request has invalid input field | Fix mapreduce fields
missing_field | *The post body was missing the "inputs" or "query" field.* | Either an inputs or query field is required | Post mpreduce request with at least one
{invalid_json, Message} | *The POST body was not valid JSON. The error from the parser was: `Message`* | Posting a mapreduce command requires correct JSON | Format mapreduce requests correctly
not_json | *The POST body was not a JSON object.* | Posting a mapreduce command requires correct JSON | Format mapreduce requests correctly


TODO: Ask bryan: do we even still use this? Or is this all legacy?
%% riak_kv_map_phase
% handle_input/3
{stop, {error, {no_candidate_nodes, exhausted_prefist, erlang:get_stacktrace(), MapperData}}, State}
% handle_info/2
{stop, {error, {dead_mapper, erlang:get_stacktrace(), MapperData}}, State}
% handle_event(mapexec_reply, State)
throw(bad_mapper_props_no_keys)

%% riak_kv_map_filter:build_filter
{error, {bad_filter, string()}}

%% riak_kv_mapred_json:parse_request
{error, {'query', Reason}}
{error, {inputs, Reason}}
{error, missing_field}
{error, {invalid_json, Message}}
{error, not_json}

% riak_kv_mapred_query:start/1
{error, bad_fetch}

%% riak_kv_mapreduce
% map_object_value
{error, notfound} % in place of a RiakObject in the mapping phase
% reduce_identity
{unhandled_entry, Other} | *Unhandled entry: `Other`* | 

% TODO: riak_kv_mrc_map
% TODO: riak_kv_mrc_pipe


%% TODO: riak_kv_console (rests heavily on riak_core)
% join()
{error, not_reachable} ->
  io:format("Node ~s is not reachable!~n", [NodeStr]),
{error, different_ring_sizes} ->
  io:format("Failed: ~s has a different ring_creation_size~n", [NodeStr]),
{error, unable_to_get_join_ring} ->
  io:format("Failed: Unable to get ring from ~s~n", [NodeStr]),
{error, not_single_node} ->
  io:format("Failed: This node is already a member of a cluster~n"),
{error, _}
  io:format("Join failed. Try again in a few moments.~n", []),
exception
  lager:error("Join failed ~p:~p", [Exception, Reason]),
  io:format("Join failed, see log for details~n"),

% leave()
{error, not_member} ->
  io:format("Failed: ~p is not a member of the cluster.~n", [node()]),
{error, only_member} ->
  io:format("Failed: ~p is the only member.~n", [node()]),
exception
  lager:error("Leave failed ~p:~p", [Exception, Reason]),
  io:format("Leave failed, see log for details~n"),

% remove/1
{error, not_member} ->
  io:format("Failed: ~p is not a member of the cluster.~n", [Node])
{error, only_member} ->
  io:format("Failed: ~p is the only member.~n", [Node])
exception:
  lager:error("Remove failed ~p:~p", [Exception, Reason]),
  io:format("Remove failed, see log for details~n"),

% down/1
{error, legacy_mode} ->
  io:format("Cluster is currently in legacy mode~n"),
{error, is_up} ->
  io:format("Failed: ~s is up~n", [Node]),
{error, not_member} ->
  io:format("Failed: ~p is not a member of the cluster.~n", [Node]),
{error, only_member} ->
  io:format("Failed: ~p is the only member.~n", [Node]),
exception:
  lager:error("Down failed ~p:~p", [Exception, Reason]),
  io:format("Down failed, see log for details~n"),

% status/1
exception:
  lager:error("Status failed ~p:~p", [Exception, Reason]),
  io:format("Status failed, see log for details~n"),

% vnode_status/1
exception:
  lager:error("Backend status failed ~p:~p", [Exception, Reason]),
  io:format("Backend status failed, see log for details~n"),

% ringready/1
{error, {different_owners, N1, N2}} ->
  io:format("FALSE Node ~p and ~p list different partition owners\n", [N1, N2]),
{error, {nodes_down, Down}} ->
  io:format("FALSE ~p down.  All nodes need to be up to check.\n", [Down]),
exception:
  lager:error("Ringready failed ~p:~p", [Exception, Reason]),
  io:format("Ringready failed, see log for details~n"),

% transfers/1
exceptions:
  lager:error("Transfers failed ~p:~p", [Exception, Reason]),
  io:format("Transfers failed, see log for details~n"),

% cluster_info/1
io:format("Cluster_info failed, permission denied writing to ~p~n", [OutFile]);
io:format("Cluster_info failed, no such directory ~p~n", [filename:dirname(OutFile)]);
io:format("Cluster_info failed, not a directory ~p~n", [filename:dirname(OutFile)]);
exception:
  lager:error("Cluster_info failed ~p:~p", [Exception, Reason]),
  io:format("Cluster_info failed, see log for details~n"),


 ----
Many of these:
"monitor `busy_dist_port` `Pid` [...{almost_current_function,...]"

 ===

Try setting zdbbl higher in `vm.args`. For example:

`+zdbbl 16384`

If network is slow, or if your're slinging large values (or
have a handful of really big ones), then normal Riak ops can cause
`busy_dist_ports`. This message means distributed erlang buffers are filling up.

If a high bandwidth networ (eg. 1 Gbit Ethernet) is congested, try setting
RTO_min down to 0 msec (if you can do that, many Linux flavors
can't do that, so use 1msec) or change the app to move less data around.

Schedulers "stuck": use the canonical check for stuckness:

[begin timer:sleep(1000),
lists:sort(tuple_to_list(erlang:statistics(run_queues))) end || _ <-
lists:seq(1,15)].

Large numeric imbalances in the output tuples indicates stuckness.
Use the "schedmon_i" daemon to automatically check & unstick
schedulers every 2.5 minutes if schedulers are getting stuck.
 ----


## riak_core

%TODO: GENERAL? 
% eg. {error, timeout}

### riak_core
Error    | Message | Description | Resolution
---------|---------|-------------|-------
self_join | | Cannot join node with itself | Join another node to form a valid cluster
not_reachable | | Cannot join unreachable node | Check your network connections, ensure erlang cookie setting `vm.args` `-setcookie`
unable_to_get_join_ring | | Cannot access cluster ring to join | Possible corrupted ring
not_single_node | | There are no other members to join | Join with at least one other node
different_ring_sizes | | The joining ring is a different size from the existing cluster ring | Don't join a node already joined to a cluster
not_member | | This node is not a member of the ring  | Cannot leave/remove/down when this is not a ring member
only_member | | This is the only member of the ring | Cannot leave/remove/down when this is the only member of the ring
is_up | | Node is expected to be down but is up | Whena node is downed, it should be down
already_leaving | *`Node` is already in the process of leaving the cluster.* | An error marking a node to leave when it is already leaving | No need to duplicate the leave command

## riak_core_app, riak_core_bucket
Error    | Message | Description | Resolution
---------|---------|-------------|-------
invalid_ring_state_dir | *Ring state directory `RingDir` does not exist, and could not be created: `Reason`* | The ring directory does not exist and no new dir can be created in expected location | Ensure that the the erlang proc can write to `ring_state_dir` or has permission to create that dir
 | *Bucket validation failed `Details`* | |
{unknown_capability, Capability} | | Attempting to use a capability unsupported by this behavior | Ensure that your `app.config` choices (eg. backends) support the behaviors you're attempting to use

### riak_core_claimant, riak_core_coverage_plan
Error    | Message | Description | Resolution
---------|---------|-------------|-------
ring_not_ready | | Ring not ready to perform command | Attempting to plan a ring change before the ring is ready to do so
legacy | | Attempting to stage a plan against a legacy ring | Staging is a feature only of Riak versions 1.2.0+
nothing_planned | | Cannot commit a plan without changes | Ensure at least one ring change is planned before running `commit`
is_claimant | | A node cannot be the claimant of it's own remove request | Remove/replace nodes from another node
already_replacement | | This node is already in the replacements request list | You cannot replace the same node twice
invalid_replacement | | A new node is currently joining from a previous operation, so a replacement request is invalid until it is no longer joining | Wait until the node is finished joining
insufficient_vnodes_available | | When creating a query coverage plan, no keyspaces are available | 

### riak_core_handoff_manager, riak_core_handoff_receiver
Error    | Message | Description | Resolution
---------|---------|-------------|-------
 | *set_recv_data called for non-existing receiver* | Cannot connect to receiver during handoff |
max_concurrency | *Handoff receiver for partition `Partition` exited abnormally after processing `Count` objects: `Reason`* | Disallow more handoff processes than the `raik_core` `handoff_concurrency` setting (defaults to 2) | If this routinely kills vnodes, this issues has been linked to leveldb compactions which can build up and block writing, which will also be accompanied by leveldb logs saying `Waiting...` or `Compacting NNN@0`
 | *An `Dir` handoff of partition `M` `I` was terminated because the vnode died* | Handoff stopped because of vnode was 'DOWN' and sender must be killed | 
 | *status_update for non-existing handoff `Target`* | Cannot get the status of a handoff `Target` module that doesn't exist | 

### riak_core_handoff_sender: start_link
Error    | Message | Description | Resolution
---------|---------|-------------|-------
 | *SSL handoff config error: property `FailProp`: `BadMat`.* | The receiver may reject the senders attempt to start a handoff 
 | *Failure processing SSL handoff config `Props`: `X`:`Y`* | 
timeout | *`Type` transfer of `Module` from `SrcNode` `SrcPartition` to `TargetNode` `TargetPartition` failed because of TCP recv timeout*
 | *`Type` transfer of `Module` from `SrcNode` `SrcPartition` to `TargetNode` `PargetPartition` failed because of `Reason`*

### riak_core_ring_handler:init/1, riak_core_ring_manager
Error    | Message | Description | Resolution
---------|---------|-------------|-------
 | *Failed to start application: `App`* | |
 | *Failed to read ring file: `Reason`* | |
 | *Failed to load ring file: `Reason`* | |
 | *ring_trans: invalid return value: `Other`* | |
 | *Error while running bucket fixup module `Fixup` from application `App` on bucket `BucketName`: `Reason`* | |
 | *Crash while running bucket fixup module `Fixup` from application `App` on bucket `BucketName` : `What`:`Why`* | |

### riak_core_stat_cache, riak_core_status, riak_core_vnode, riak_core_vnode_manager, riak_core_vnode_worker_pool
Error    | Message | Description | Resolution
---------|---------|-------------|-------
{not_registered, App} | | |
{different_owners, N1, N2} | | |
{nodes_down, Down} | | |
vnode_exiting | *`Mod` failed to store handoff obj: `Err`* | | |
 | *`Index` `Mod` worker pool crashed `Reason`* | | |
 | *Received xfer_complete for non-existing repair: `ModPartition`* | | |
vnode_shutdown | | | 

% TODO: riak_core_console

## riak_sysmon

NONE

 
## poolboy

% poolboy
%% unknown event sent to module
{error, invalid_message}

 
## riak_pb

TODO: search error, fail, lager:error, lager:critical or throw

% TODO: list out the standard error response messages from the API

NONE

