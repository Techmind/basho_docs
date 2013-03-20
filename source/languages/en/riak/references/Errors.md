---
title: Common Errors List
project: riak
version: 1.3.0+
document: reference
toc: true
audience: advanced
keywords: [errors]
---

## riak_kv

Many KV errors have prescriptive messages. For example, the mapreduce parse_input phase will respond thusly with an invalid input:
"Inputs must be a binary bucket, a tuple of bucket and key-filters, a list of target tuples, or a search, index, or modfun tuple: INPUT".
For these cases we leave it to Riak to explain the correct course of action. For the remaining common error codes, they are often marked by Erlang atoms. This document describes those terse error codes.

Crash on Startup
CRASH REPORT
INFO REPORT

### riak_client

Error    | Description | Resolution
---------|-------------|-------
notfound | No value found | Value was deleted, or was not yet stored or replicated
timeout | The given action took too long to reply | Check network settings; ensure remote nodes are running and reachable; ensure all nodes have the same erlang vm `-name`
too_many_fails | Too many write failures to satisfy W or DW | Try write again; ensure your nodes/network is healthy; set a lower W or DW value 
{n_val_violation, N} | (W > N) or (DW > N) or (PW > N) or (R > N) or (PR > N) | No W or R values may be greater than N
{r_val_unsatisfied, R, Replied} | Not enough nodes replied to satisfy the R value, contains the given `R` value and the actual number of `Replies` | Too many nodes are down or the R value was set too high
{pr_val_unsatisfied, PR, Primaries} | Same as `r_val_unsatisfied` but only counts Primary node replies | Too many primary nodes are down or the PR value was set too high
{r_val_violation, R} | The given R value was non-numeric and not a valid setting (on, all, quorom) | Set a valid R value
{pr_val_violation, R} | Same as `r_val_violation` but concerning Primary reads | Set a valid PR value
{rw_val_violation, RW} | The given RW property was non-numeric and not a valid setting (on, all, quorom) | Set a valid RW value
{w_val_violation, W} | The given W property was non-numeric and not a valid setting (on, all, quorom) | Set a valid W value
{pw_val_violation, PW} | Same as `w_val_violation` but concerning Primary writes | Set a valid PW value
{dw_val_violation, DW} | Same as `w_val_violation` but concerning durable writes | Set a valid DW value
{insufficient_vnodes, NumVnodes, need, R} | R was set greater than the total VNodes | Set a proper R value
{deleted, Vclock} | The value was already deleted, includes the current vector clock | Riak will eventually clean up this tombstone
{bad_qterm, QueryTerm} | Bad query when performing mapreduce | Fix your mapreduce query
{invalid_inputdef, InputDef} | Bad `inputs` definitions when running mapreduce | Fix `inputs` settings; Set `mapred_system` from `legacy` to `pipe`
key_too_large | The key was larger than 65536 bytes | Use a smaller key


### riak_index, riak_kv_pb_index

Error    | Message | Description | Resolution
---------|---------|-------------|-------
{unknown_field_type, Field} | "Unknown field type for field: '`Field`'." | Unknown index field extention (begins with underscore) | The only value field types are `_int` and `_bin`
{field_parsing_failed, {Field, Value}} | "Could not parse field '`Field`', value '`Value`'." | Could not parse an index field | Most commonly an _int field which cannot be parsed
{siblings_not_allowed, Object} | "Siblings not allowed: `Object`" | The hook to index cannot abide siblings | Set the buckets `allow_mult` property to false
{not_supported, mapred_index, FlowPid} | | Index lookups for mapreduce are only supported with Pipe | Set `mapred_system` from `legacy` to `pipe`
{invalid_range, Args} | | Index range query has `Start` > `End` | Fix your query
{too_few_arguments, Args} | | Index query requires at least one argument | Fix your query format
{too_many_arguments, Args} | | Index query is malformed with more than 1 (exact) or 2 (range) values | Fix your query format
 | "Invalid equality query `SKey`" | Equality query is required and must be binary for an index call | Pass in an equality value to an 
 | "Invalid range query: `Min` -> `Max`" | Both range query values are required and must be binary an index call | Pass in both range values


### kv_app & backup

Error    | Message | Description | Resolution
---------|---------|-------------|-------
invalid_storage_backend | "storage_backend `Backend` is non-loadable." | Invalid backend choice when starting up Riak | Set a valid backend in `app.config` (eg. *{storage_backend, riak_kv_bitcask_backend}*)
{could_not_reach_node, Node} | | Erlang process was not reachable | Check network settings; ensure remote nodes are running and reachable; ensure all nodes have the same erlang vm `-name`
{nodes_not_synchronized, Members} | | Rings of all members are not synchronized | ???


### riak_kv_multi_backend
Error    |  Description | Resolution
---------|--------------|-------
multi_backend_config_unset | No configuration for multi backend | Configure at least one backend under `multi_backend` in `app.config`
{invalid_config_setting, multi_backend, list_expected} | Multi backend configuration requires a list | Wrap `multi_backend` config value in a list
{invalid_config_setting, multi_backend, list_is_empty} | Multi backend configuration requires a value | Configure at least one backend under `multi_backend` in `app.config`
{invalid_config_setting, multi_backend_default, backend_not_found} | | Must choose a valid backend type to configure
{riak_kv_multi_backend, undefined_backend, BackendName} | Bakend defined for a bucket is invalid | Define a valid backed before using this bucket

### riak_kv_bitcask_backend
Error    | Message | Description | Resolution
---------|---------|-------------|-------
data_root_unset | "Failed to create bitcask dir: data_root is not set" | The `data_root` config setting is required | Set `data_root` as the base directory where to store bitcask data, under the `bitcask` section
data_root_not_set | | Same as `data_root_unset` | Set the `data_root` directory in config

### riak_kv_eleveldb_backend
Error    | Message | Description | Resolution
---------|---------|-------------|-------

### riak_kv_memory_backend

Error    | Message | Description | Resolution
---------|---------|-------------|-------
reset_disabled | | Attempted to reset the memory backend in production | Don't use this in production

### Javascript

riak_kv_js_manager, riak_kv_js_vm

Error    | Message | Description | Resolution
---------|---------|-------------|-------
no_vms | "JS call failed: All VMs are busy." | All Javascript VMs are in use | Wait and run again; Increase Javascript VMs in `app.config` (`map_js_vm_count`, `reduce_js_vm_count`, or `hook_js_vm_count`)
bad_utf8_character_code | "Error JSON encoding arguments: `Args`" | A UTF-8 character give was a bad format | Only use correct UTF-8 characters for Javascript code and arguments
bad_json | | Bad JSON formatting | Only use correctly formatted JSON for Javascript command arguments
{format, Reason} | "Invalid bucket properties: `Details`" | Listing bucket properties will fail if invalid | Fix bucket properties


### riak_kv_pb_mapred
Error    | Message | Description | Resolution
---------|---------|-------------|-------
{unknown_content_type, ContentType} | | Bad content type for mapreduce query | Only application/json and application/x-erlang-binary are accepted
 | "Phase `Fitting`: `Reason`" | A bad argument or pipe configuration | 
bad_mapred_inputs | | A bad value sent to mapreduce |
javascript_reduce_timeout | | Javascript reduce function taking too long | For large numbers of objects, your JavaScript functions may become bottlenecks. Decrease the quantity of values being passed to and returned from the reduce functions, or rewrite as Erlang functions

### riak_kv_put_core
Error    | Description | Resolution
---------|-------------|-------
{w_val_unsatisfied, RepliesW, RepliesDW, W, DW} | Not enough nodes replied to satisfy the W value, contains the given `W` value and the actual number of `Replies` for either W or DW | Too many nodes are down or the W or DW value was set too high
{pw_val_unsatisfied, PR, Primaries} | Same as `w_val_unsatisfied` but only counts Primary node replies | Too many primary nodes are down or the PW value was set too high

### riak_kv_put_fsm
Error    | Message | Description | Resolution
---------|---------|-------------|-------
all_nodes_down | | No nodes are available | Add nodes to the Ring
{coord_handoff_failed, Reason} | "Unable to forward put for `Key` to `CoordNode` - `Reason`" |
precommit_fail | "Pre-commit hook `Mod`:`Fun` failed with reason `Reason`" | The given precommit function failed for the given reason | Fix the precommit function code
local_put_failed | | A local vnode PUT operation failed | Try put again
{hook_crashed, {Mod, Fun, Class, Exception}} | Precommit hook throws EXIT | Fix the precommit function code
{invalid_return, {Mod, Fun, Result}} | "Problem invoking pre-commit hook `Mod`:`Fun`, invalid return `Result`" | The given precommit function gave an invalid return for the given Result | Ensure the function returns a valid result
{invalid_hook_def, HookDef} | "Invalid post-commit hook definition `Def`" | No Erlang `mod` and `fun` or javascript function `name` | Define the hook with the correct settings


### riak_kv_vnode
Error    | Message | Description | Resolution
---------|---------|-------------|-------
 | "Failed to start `Mod` Reason: `Reason`" | Riak KV failed to start for given `Reason` | Several possible reasons for failure, read the attached reason for insight into resolution
{indexes_not_supported, Mod} | | The chosen backend does not support indexes (only eleveldb currently support 2i) | Set `app.config` to use the eLevelDB backend
receiver_down | | Remote process failed to acknowledge request | Can occur when listkeys is called

%% TODO: Worry about WM at all?

### riak_kv_wm_mapred

Error    | Message | Description | Resolution
---------|---------|-------------|-------
 | "riak_kv_w_reduce requires a function as argument, not a `Type`" | Reduce requires a function object, not any other type | This shouldn't happen
{'query', Reason} | "An error occurred parsing the \"query\" field." | Mapreduce request has invalid query field | Fix mapreduce query
{inputs, Reason} | "An error occurred parsing the \"inputs\" field." | Mapreduce request has invalid input field | Fix mapreduce fields
missing_field | "The post body was missing the \"inputs\" or \"query\" field." | Either an inputs or query field is required | Post mpreduce request with at least one
{invalid_json, Message} | "The POST body was not valid JSON. The error from the parser was: `Message` | Posting a mapreduce command requires correct JSON | Format mapreduce requests correctly
not_json | "The POST body was not a JSON object." | Posting a mapreduce command requires correct JSON | Format mapreduce requests correctly


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
{unhandled_entry, Other} | "Unhandled entry: `Other`" | 

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
`busy_dist_ports`.

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
not_reachable | | Cannot join unreachable node | Check your network connections, ensure erlang `-name` are the same in `vm.args`,
unable_to_get_join_ring | | Cannot access cluster ring to join | Possible corrupted ring
not_single_node | | There are no other members to join | Join with at least one other node
different_ring_sizes | | The joining ring is a different size from the existing cluster ring | Don't join a node already joined to a cluster
not_member | | This node is not a member of the ring  | Cannot leave/remove/down when this is not a ring member
only_member | | This is the only member of the ring | Cannot leave/remove/down when this is the only member of the ring
is_up | | Node is expected to be down but is up | Whena node is downed, it should be down
already_leaving | "`Node` is already in the process of leaving the cluster." | An error marking a node to leave when it is already leaving | No need to duplicate the leave command

## riak_core_app, riak_core_bucket
Error    | Message | Description | Resolution
---------|---------|-------------|-------
invalid_ring_state_dir | "Ring state directory `RingDir` does not exist, and could not be created: `Reason`" | The ring directory does not exist and no new dir can be created in expected location | Ensure that the the erlang proc can write to `ring_state_dir` or has permission to create that dir
 | "Bucket validation failed `Details`" | |
{unknown_capability, Capability} | Attempting to use a capability unsupported by this behavior | Ensure that your `app.config` choices (eg. backends) support the behaviors you're attempting to use

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
% set_recv_data/2
 | "set_recv_data called for non-existing receiver" | |
% send_handoff/2
max_concurrency | | |
% status_update/2
 | "status_update for non-existing handoff `ModSrcTarget`" | |
% handle_info/’DOWN'
 | "An `Dir` handoff of partition `M` `I` was terminated for reason: `Reason`" | |
% handle_info({tcp
 | "Handoff receiver for partition `Partition` exited abnormally after processing `Count` objects: `Reason`" | |

### riak_core_handoff_sender: start_link
%% The receiver may reject the senders attempt to start a handoff.
exit({shutdown, max_concurrency})
exit({shutdown, timeout})
lager:error("SSL handoff config error: property ~p: ~p.", [FailProp, BadMat]),
lager:error("Failure processing SSL handoff config ~p: ~p:~p", [Props, X, Y])
lager:error("~p transfer of ~p from ~p ~p to ~p ~p failed because of TCP recv timeout”,  [Type, Module, SrcNode, SrcPartition, TargetNode, TargetPartition] ++ Args)).
lager:error("~p transfer of ~p from ~p ~p to ~p ~p failed because of ~p”,  [Type, Module, SrcNode, SrcPartition, TargetNode, TargetPartition, Reason)).

### riak_core_ring_handler:init/1, riak_core_ring_manager
lager:critical("Failed to start application: ~p", [App]),
% reload_ring(live/
lager:critical("Failed to read ring file: ~p", [lager:posix_error(Reason)]),
lager:critical("Failed to load ring file: ~p", [lager:posix_error(Reason)])
throw({error, Reason})
% handle_call({ring_trans
lager:error("ring_trans: invalid return value: ~p", [Other]),
lager:error("Error while running bucket fixup module ~p from application ~p on bucket ~p: ~p", [Fixup, App, BucketName, Reason]),
lager:error("Crash while running bucket fixup module ~p from application ~p on bucket ~p : ~p:~p", [Fixup, App, BucketName, What, Why]),

### riak_core_stat_cache
% handle_call({get_stats
{error, {not_registered, App}}

### riak_core_status
% ring_ready/0
{error, {different_owners, N1, N2}}
{error, {nodes_down, Down}}

### riak_core_vnode, riak_core_vnode_manager
% handle_sync_event({handoff_data
vnode_exiting | "`Mod` failed to store handoff obj: `Err`"
% handle_info({‘EXIT’
 | "`Index` `Mod` worker pool crashed `Reason`"
% handle_call({xfer_complete
 | "Received xfer_complete for non-existing repair: `ModPartition`"


%% riak_core_vnode_worker_pool
% shutdown({work
{error, vnode_shutdown}

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

