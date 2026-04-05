# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Repository

Tdarr flow definitions for the home media server. Files use `.yml` extensions but contain **JSON** (not YAML).

Tdarr runs on `hl15-beast`, accessible via `ssh root@hl15-beast` and at `https://tdarr.nas.home.thecrimsontint.com`.

## Applying Flow Updates

**Do not ask the user to reimport flows manually.** After editing a flow file, apply it directly via the Tdarr cruddb API:

```bash
# Update an existing flow (replace <ID> with the flow's _id field)
FLOW=$(cat 'path/to/flow.yml') && \
curl -sk -X POST "https://tdarr.nas.home.thecrimsontint.com/api/v2/cruddb" \
  -H "Content-Type: application/json" \
  -d "{\"data\":{\"collection\":\"FlowsJSONDB\",\"mode\":\"update\",\"docID\":\"<ID>\",\"obj\":$FLOW}}"
```

To get the `_id` for a flow:
```bash
node -e "const d=JSON.parse(require('fs').readFileSync('path/to/flow.yml','utf8')); console.log(d._id, d.name);"
```

To verify the update was applied:
```bash
curl -sk -X POST "https://tdarr.nas.home.thecrimsontint.com/api/v2/cruddb" \
  -H "Content-Type: application/json" \
  -d '{"data":{"collection":"FlowsJSONDB","mode":"getAll"}}' | python3 -c "
import sys, json
for f in json.load(sys.stdin):
    print(f['_id'], f['name'], '- plugins:', len(f.get('flowPlugins',[])), 'edges:', len(f.get('flowEdges',[])))
"
```

A successful update returns HTTP 200 with an empty body.

## Plugin Directory

Plugins are under `/mnt/data-nvme/media-mgmt/tdarr/server/Tdarr/Plugins/FlowPlugins/CommunityFlowPlugins/` on hl15-beast. Use SSH to verify plugin names, versions, and input schemas before adding new plugin nodes to a flow.

## Node Tags

The `hl15-beast` Tdarr node should be tagged `mapped` in the Node Options panel. Unmapped nodes (e.g. `crimbedroom`, `crimhtpc` — Windows machines) do not have direct filesystem access to the NAS media library, so the `replaceOriginalFile` plugin must only run on the mapped node. Use `tagsWorkerType` with `requiredNodeTags: mapped` to gate this step.
