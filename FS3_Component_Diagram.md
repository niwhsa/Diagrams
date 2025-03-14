``` mermaid
graph TD
    subgraph "User Space"
        App[Application]
        FUSE[FUSE Interface]
    end

    subgraph "3FS Components"
        RP[Request Processor]
        RH[Request Handler Registry]
        CM[Cache Manager]
        BA[Block Allocator]
        IO[IO Service]
        MD[Metadata Service]
        
        subgraph "Storage Layer"
            SN[Storage Nodes]
            FDB[(FoundationDB)]
        end
    end

    %% Read Flow
    App -->|"1/ read()"| FUSE
    FUSE -->|2/ FUSE Request| RP
    RP -->|3/ Lookup Handler| RH
    RP -->|4/ Check Cache| CM
    CM -->|5a/ Cache Hit| RP
    CM -->|5b/ Cache Miss| IO
    IO -->|6. Read Request| SN
    RP -->|7. Get Metadata| MD
    MD -->|8. Fetch| FDB

    %% Write Flow
    App -->|"1. write()"| FUSE
    FUSE -->|2. Write Request| RP
    RP -->|3. Get Handler| RH
    BA -->|4. Allocate Blocks| IO
    IO -->|5. Write Data| SN
    MD -->|6. Update Metadata| FDB
    CM -->|7. Invalidate Cache| RP
```
