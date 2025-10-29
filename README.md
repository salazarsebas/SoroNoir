# SoroNoir


### A Rust Crate for Seamless Noir ZK-Proof Integration on Stellar Soroban

SoroNoir is envisioned as a specialized Rust crate that bridges the Noir domain-specific language (DSL) for zero-knowledge proofs with Stellar's Soroban smart contract platform. Drawing inspiration from Garaga—an open-source tool for generating Cairo verifiers from Noir circuits on Starknet—SoroNoir aims to provide a more streamlined, Stellar-optimized alternative. While Garaga excels in automating elliptic curve operations (e.g., BN254 and BLS12-381 pairings), proof generation via the "ultra keccak honk" backend, and on-chain deployment for Starknet, SoroNoir will adapt these concepts to Soroban's Rust-based ecosystem. The goal is to enable developers to easily compile Noir circuits, generate proofs off-chain, and deploy verifiable smart contracts on Stellar, with a focus on privacy-enhancing applications such as shielded transactions and identity verification.

This design builds upon the proof-of-concept demonstrated in your noir-stellar repository, which successfully validated the compatibility of Barretenberg, Noir, and Stellar by compiling a simple private sum circuit, generating proofs, and verifying them on-chain. However, SoroNoir elevates this from a basic demonstration to a robust, user-friendly toolset, emphasizing ease of use, extensibility, and performance optimizations tailored to Stellar's low-cost, high-throughput environment.

#### Key Requirements
To ensure SoroNoir is functional, maintainable, and aligned with best practices, the following requirements are essential:

1. **Core Dependencies**:
   - **Rust Ecosystem**: Target Rust 1.70 or later for compatibility with Soroban's WebAssembly (WASM) compilation. The crate must support the `wasm32-unknown-unknown` target to produce deployable Soroban contracts.
   - **Soroban SDK**: Include `soroban-sdk` (version 20.5.0 or higher) for building and interacting with Soroban smart contracts, handling types like `BytesN` for proof data and storage operations.
   - **Cryptographic Libraries**: Rely on `ark-bn254`, `ark-ec`, and `ark-ff` (version 0.4.0 series) as fallbacks for elliptic curve operations, including pairings and point arithmetic. These are critical for verifying zk-SNARKs or zk-STARKs when native Soroban host functions are unavailable or insufficient.
   - **CLI and Utility Crates**: Use `clap` (version 4.5.0) for a command-line interface that simplifies workflows, `serde` and `serde_json` for serializing proofs and verification keys, and `hex` for encoding/decoding data in a Stellar-compatible format.
   - **Development Dependencies**: Incorporate `assert_cmd` for testing CLI commands, ensuring reliability without requiring full end-to-end simulations.

2. **External Tools Integration**:
   - **Noir CLI (nargo)**: Leverage nargo for circuit compilation into Arithmetic Circuit Intermediate Representation (ACIR), generating proving and verification keys. This must be installed via Cargo (`cargo install nargo`).
   - **Barretenberg**: As the primary backend for Noir, use Barretenberg for efficient proof generation and verification key handling. Installation involves cloning the AztecProtocol/barretenberg repository and building it, focusing on the "honk" flavor for keccak-based optimizations.
   - **Stellar CLI**: Utilize stellar-cli for building, deploying, and invoking Soroban contracts on Stellar networks (e.g., testnet or mainnet). Commands such as `stellar contract build`, `stellar contract deploy`, and `stellar contract invoke` will be wrapped in the crate's CLI to automate workflows.
   - **No Additional Installations**: Avoid dependencies on external package managers beyond Cargo, ensuring the crate remains lightweight and portable.

3. **System and Environment Requirements**:
   - **Operating System**: Compatible with Linux, macOS, or Windows (via WSL), with a focus on environments supporting WASM compilation.
   - **Hardware**: Moderate CPU and memory (at least 4GB RAM) for off-chain proof generation, as Barretenberg operations can be computationally intensive.
   - **Network Access**: Required for deploying to Stellar testnet/mainnet, but not for local development or proof generation.
   - **Licensing and Compliance**: Release under MIT license to encourage community contributions, with clear documentation on security considerations for zk-proof systems.

#### High-Level Design and Functionality
SoroNoir's architecture emphasizes modularity, drawing from Garaga's strengths in automation while addressing Stellar-specific constraints like compute budgets (typically 100k-500k instructions per transaction). The crate operates in a hybrid model: off-chain for resource-heavy tasks (circuit compilation and proof generation) and on-chain for lightweight verification.

- **Circuit Compilation and Verifier Generation**: Users provide a Noir circuit directory. SoroNoir invokes nargo to compile the circuit and Barretenberg to produce verification keys. It then generates a templated Rust Soroban contract, customizing elements like the verifier struct with ark-based pairing checks. This improves on Garaga by including auto-optimization for WASM size and fallback mocks for development.

- **Proof Generation**: Off-chain proof creation, using Barretenberg to compute proofs from private inputs (e.g., via Prover.toml). Outputs are serialized in JSON/hex formats for easy invocation, enhancing usability with error-handling and progress indicators absent in basic Garaga flows.

- **Deployment and Invocation Workflow**: The CLI streamlines Stellar interactions: build the WASM artifact, deploy the contract with verification keys, and invoke verification methods. For example, it handles public inputs and proof data as arguments, ensuring seamless integration with Stellar's Horizon API for transaction monitoring.

- **Enhancements Over Garaga**: SoroNoir introduces built-in templates for common privacy circuits (e.g., range proofs for asset amounts), support for recursive proofs to manage compute limits, and detailed logging for debugging. It also provides warnings for potential budget overruns, suggesting aggregations or oracles—features that extend Garaga's Starknet-focused optimizations to Stellar's unique ledger model.

#### Limitations and Mitigation Strategies
As initial POC, Soroban's current lack of native elliptic curve primitives poses a challenge; ark libraries serve as a bridge but may increase contract size. To mitigate, SoroNoir includes feature flags for mock verifications during testing and plans integration with upcoming Soroban updates (e.g., Protocol 24 enhancements). Security audits are recommended for production use, focusing on proof soundness and resistance to side-channel attacks.

#### Implementation Roadmap
Begin by initializing the crate with `cargo new --bin soronoir`. Populate the structure with the outlined components, prioritizing CLI commands for generation and deployment. Test iteratively on Stellar testnet, incorporating feedback from the Stellar community. Once stable, publish to crates.io.

