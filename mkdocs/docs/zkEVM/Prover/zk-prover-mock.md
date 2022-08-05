<!-- The mock server is at git@github.com:hermeznetwork/zk-mock-prover.git-->

# Server Definition
- `proto/zk-prover-proto` contains the service specifications.

    syntax = "proto3";

    package zkprover;

    service ZKProver {
        rpc GetStatus(NoParams) returns (State) {}
        rpc GenProof(stream Batch) returns (stream Proof) {}
        rpc Cancel(NoParams) returns (State) {}
        rpc GetProof(NoParams) returns (Proof) {}
    }

    message NoParams {}

    message State {
        enum Status {
            IDLE = 0;
            ERROR = 1;
            PENDING = 2;
            FINISHED = 3;
        }
        Status status = 1;
        Proof proof = 2;
    }

    message ProofX {
        repeated string proof = 1;
    }

    message PublicInputs {
        bytes currentStateRoot = 1;
        bytes currentLocalExitRoot = 2;
        bytes newStateRoot = 3;
        bytes newLocalExitRoot = 4;
        string sequencerAddress = 5;
        bytes l2TxsDataLastGlobalExitRoot = 6;
        uint64 chainId = 7;
    }

    message Proof {
        repeated string proofA = 1;
        repeated ProofX proofB = 2;
        repeated string  proofC = 3;
        PublicInputs publicInputs = 4;
    }

    message Batch {
        string message = 1;
        bytes currentStateRoot = 2;
        bytes newStateRoot = 3;
        bytes l2Txs = 4;
        bytes lastGlobalExitRoot = 5;
        string sequencerAddress = 6;
        uint64 chainId = 7;
    }

- Following documentation explains its behaviour further:

## Service Functionalities
```
rpc GetStatus(NoParams) returns (State) {}
rpc GenProof(stream Batch) returns (stream Proof) {}
rpc Cancel(NoParams) returns (State) {}
rpc GetProof(NoParams) returns (Proof) {}
```

### GetStatus
This function lets us know the status of the prover.

The client does not need to enter data to make this call.
The status is returned in the following form:
```
message State {
    enum Status {
        IDLE = 0;
        ERROR = 1;
        PENDING = 2;
        FINISHED = 3;
    }
    Status status = 1;
    Proof proof = 2;
}
```

The status will be one of those defined in the `enum`. Proof is only defined if the status is `FINISHED`.

### GenProof
This function generates the proofs.

The client must provide the following information to the server when calling the function:
```
message Batch {
    string message = 1;
    bytes currentStateRoot = 2;
    bytes newStateRoot = 3;
    bytes l2Txs = 4;
    bytes lastGlobalExitRoot = 5;
    string sequencerAddress = 6;
    uint64 chainId = 7;
}
```

where the message can be:
- `"calculate"`: to generate the proof
- `"cancel"`: to cancel the last proof

and the server will respond:
```
message Proof {
    repeated string proofA = 1;
    repeated ProofX proofB = 2;
    repeated string  proofC = 3;
    PublicInputs publicInputs = 4;
}
```

where:
```
message PublicInputs {
    bytes currentStateRoot = 1;
    bytes currentLocalExitRoot = 2;
    bytes newStateRoot = 3;
    bytes newLocalExitRoot = 4;
    string sequencerAddress = 5;
    bytes l2TxsDataLastGlobalExitRoot = 6;
    uint64 chainId = 7;
}

message ProofX {
    repeated string proof = 1;
}
```

This channel will remain open until client decides to close it. In this way, the client can continue requesting proofs by sending the message `Batch`.

### Cancel
If the previous channel is closed and the server has computed a proof, the client can cancel it with this call.

The client does not need to enter data to make this call.
The prover returns the status to confirm that the proof calculation is cancelled.

### GetProof
This function gets the last calculated proof.

The client does not need to enter data to make this call.
If the status is `FINISHED`, the last proof is returned.