# ZKsync Era verification key generation

If an upgrade changes the ZK circuit, the verifier contract is updated as well. To confirm that the newly deployed verifier matches the circuit’s verification keys, regenerate the verification keys locally, construct the Verifier contract, and compare its bytecode to what’s deployed on-chain. This guide focuses on generating the keys and the verifier contract.

1) Clone the zksync-era repo and check out the branch where the circuit is patched:

```
git clone https://github.com/matter-labs/zksync-era.git
```

2) Run the verification key generation command. Generating verification keys is a heavy task and may take ~15 minutes on a modern laptop.

```
ZKSYNC_USE_CUDA_STUBS=true cargo +nightly-2025-03-19 run --manifest-path prover/crates/bin/vk_setup_data_generator_server_fri/Cargo.toml --release --bin key_generator -- generate-vk --path prover/data/keys
```

3) Generate the verifier contracts.

```
cd contracts/tools
# Copy keys to the verifier generator tool
cp ../../prover/data/keys/fflonk_verification_snark_key.json data/fflonk_scheduler_key.json
cp ../../prover/data/keys/verification_snark_key.json data/plonk_scheduler_key.json
# Run the tool to generate verifiers
cargo run --bin zksync_verifier_contract_generator --release -- --plonk_input_path data/plonk_scheduler_key.json --fflonk_input_path data/fflonk_scheduler_key.json --plonk_output_path ../l1-contracts/contracts/state-transition/verifiers/L1VerifierPlonk.sol --fflonk_output_path ../l1-contracts/contracts/state-transition/verifiers/L1VerifierFflonk.sol
```
