# Alex playing around with local first

The core ideas

- One SQLite database per workspace that exists on the remote server
- Each user has a local database with the same schema
- The local db schema is synced with the remote schema based on what the user has access to
- The remote db has an extensive list of "commands" that is an exhaustive list of all actions that has been taken by all users

## Commands

```mermaid
sequenceDiagram
    participant RDB as Remote DB
    participant CS as Command Stream
    participant LDB1 as Local DB (User 1)
    participant LDB2 as Local DB (User 2)

    Note over RDB,LDB2: Initial Setup
    RDB->>LDB1: Sync schema
    RDB->>LDB2: Sync schema

    Note over RDB,LDB2: Ongoing Sync Process
    RDB->>CS: Broadcast new commands
    CS->>CS: Filter and sort commands
    CS->>LDB1: Send relevant commands (if user has access)
    CS->>LDB2: Send relevant commands (if user has access)
    LDB1->>LDB1: Apply commands
    LDB2->>LDB2: Apply commands
    Note over LDB1,CS: User 1 makes changes
    LDB1->>LDB1: Update local DB
    LDB1->>CS: Send new command
    CS->>RDB: Update Remote DB
    CS->>LDB2: Send command to User 2
    LDB2->>LDB2: Update local DB

    Note over CS,LDB2: User 2 added to a channel or a new document
    CS->>LDB2: Send "User 2 added to channel" command
    LDB2->>LDB2: Recognize need for full sync
    LDB2->>RDB: Request full sync of channel
    RDB->>LDB2: Send complete channel data
    LDB2->>LDB2: Perform full sync of channel

```
