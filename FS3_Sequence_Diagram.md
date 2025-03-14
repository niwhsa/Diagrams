``` mermaid
sequenceDiagram
    participant App as Application
    participant FI as FUSE Interface
    participant RP as Request Processor
    participant RH as Request Handler
    participant CM as Cache Manager
    participant BA as Block Allocator
    participant IO as IO Service
    participant MD as Metadata Service
    participant SN as Storage Nodes
    participant FDB as FoundationDB

    %% Read Sequence
    Note over App,FDB: Read Operation
    App->>FI: read(path, size, offset)
    FI->>RP: process_fuse_request()
    RP->>RH: get_handler(FUSE_READ)
    RP->>CM: get(path, offset, size)
    alt Cache Hit
        CM-->>RP: return cached_data
    else Cache Miss
        RP->>MD: get_metadata(path)
        MD->>FDB: fetch_metadata()
        FDB-->>MD: metadata
        MD-->>RP: block_locations
        RP->>IO: submit_read(block_locations)
        IO->>SN: read_blocks()
        SN-->>IO: block_data
        IO-->>RP: read_result
        RP->>CM: put(path, data)
    end
    RP-->>FI: send_response()
    FI-->>App: return data

    %% Write Sequence
    Note over App,FDB: Write Operation
    App->>FI: write(path, data, size)
    FI->>RP: process_fuse_request()
    RP->>RH: get_handler(FUSE_WRITE)
    RP->>MD: begin_transaction()
    RP->>BA: allocate(size)
    BA-->>RP: block_allocations
    RP->>IO: submit_write(blocks)
    IO->>SN: write_blocks()
    SN-->>IO: write_complete
    IO-->>RP: write_result
    RP->>MD: update_metadata(path)
    MD->>FDB: commit_transaction()
    RP->>CM: invalidate(path)
    RP-->>FI: send_response()
    FI-->>App: return status
```
