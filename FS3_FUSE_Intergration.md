```mermaid
graph TD
    subgraph "FUSE Daemon"
        FD[FUSE Daemon]
        RP[Request Processor]
        PH[Protocol Handler]
    end
    
    subgraph "3FS Components"
        CM[Cache Manager]
        BA[Block Allocator]
        IO[IO Service]
        MD[Metadata Service]
    end
    
    K[Kernel FUSE] -->|Requests| FD
    FD -->|Dispatch| RP
    RP -->|Translate| PH
    
    PH -->|Cache Operations| CM
    PH -->|Block Operations| BA
    PH -->|IO Operations| IO
    PH -->|Metadata Operations| MD
    
    PH -->|Response| FD
    FD -->|Reply| K
```
