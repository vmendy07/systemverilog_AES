# SystemVerilog: Advanced Encryption Standard (AES)

## Description

This project is an implementation of the AES in CTR mode using SystemVerilog.

AES (Advanced Encryption Standard) is a symmetric encryption algorithm established by the U.S. National Institute of Standards and Technology (NIST) in 2001, known for its security, efficiency, and flexibility with key sizes of 128, 192, or 256 bits. It is widely used for securing data in various applications, from government communications to everyday internet transactions.

![AES](attachments/README/aes_structure.jpg)

CTR (Counter) mode is an encryption method that transforms plaintext blocks into ciphertext blocks by XORing them with a keystream generated from a block cipher. It is a stream cipher that provides parallel processing and random access capabilities, making it suitable for applications that require high throughput and low latency.

![CTR encryption](attachments/README/CTR_encryption.png)
![CTR decryption](attachments/README/CTR_decryption.png)

## Design

### Module hierarchy

![Hierarchy](attachments/README/Hierarchy.png)

### [AesCtr module](AES.srcs/sources_1/new/AesCTR.sv)

![AesCtr module](attachments/README/AesCtr_module.png)

The `AesCtr` module is the top-level module that instantiates the `AesCore` module to perform the AES encryption and decryption operations in CTR mode using an FSM with the following states:

- `IDLE`: The initial state where the module waits for the `load` or the `start` signal to be asserted. When `load` is asserted, the module stores the `key` and `iv` values and transitions to the `KEY_EXPANSION` state. When `start` is asserted, the module transitions to the `RUNNING` state.
- `KEY_EXPANSION`: The state where the module waits for the `AesCore` module to generate the round keys. When the round keys are generated (i.e., `AesCore`'s `idle` signal is asserted), the module returns to the `IDLE` state.
- `RUNNING`: The state where the module passes the `counter` value to the `AesCore` module to generate the keystream. When the keystream is generated (i.e., `AesCore`'s `idle` signal is asserted), the module increments the `counter` and returns to the `IDLE` state.

![AesCtr FSM](attachments/README/AesCtr_FSM.png)

You have to load the `key` and `iv` (initialization vector) before starting the encryption or decryption process. The `key` and `iv` are loaded when the `load` signal is asserted in 1 cycle. To make the module work correctly, you must load the `key` and `iv` when the module is in the `IDLE` state.

The output of the `AesCtr` (i.e., `oBlock`) is the XORed result of the input block (`iBlock`) and the keystream generated by the `AesCore` module, which is only valid when the `idle` signal is asserted.

### [AesCore module](AES.srcs/sources_1/new/AesCore.sv)

![AesCore module](attachments/README/AesCore_module.png)

The `AesCore` module is the core module that performs the AES encryption and decryption operations using `KeyExpansion`, `Cipher`, and `InvCipher` modules. It uses the `encrypt` signal to select between encryption and decryption operations. When `encrypt` is asserted, the module performs the encryption operation; otherwise, it performs the decryption operation. The `idle` signal is asserted when the operation is completed.

The `AesCore` module has an FSM with the following states:

- `IDLE`: The initial state where the module waits for the `load` or the `start` signal to be asserted. When `load` is asserted, the module stores the `key` value in its internal register (otherwise, the `key` must be stable for several cycles to calculate the round keys), starts the `KeyExpansion` module to generate the round keys, and transitions to the `KEY_EXPANSION` state. When `start` is asserted, the module starts the `Cipher` and `InvCipher` modules to perform the encryption or decryption operation and transitions to the `RUNNING` state.
- `KEY_EXPANSION`: The state where the module waits for the `idle` signal of the `KeyExpansion` module to be asserted. The module returns to the `IDLE` state when the `idle` of the `KeyExpansion` module is asserted.
- `RUNNING`: The state where the module waits for the `idle` signal of the `Cipher` or `InvCipher` (depending on the `encrypt` signal) module to be asserted. When `idle` is asserted, the module transitions back to the `IDLE` state.

![AesCore FSM](attachments/README/AesCore_FSM.png)

### [KeyExpansion module](AES.srcs/sources_1/new/KeyExpansion.sv)

![KeyExpansion module](attachments/README/KeyExpansion_module.png)

The `KeyExpansion` module generates the round keys from the input `key`. The diagram below shows the key expansion FMS:

![KeyExpansion FMS](attachments/README/KeyExpansion_FMS.png)

> **Note:** You can find the Pseudocode in chapter 5.2 of the [AES specification](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197-upd1.pdf).

### [Cipher](AES.srcs/sources_1/new/Cipher.sv) and [InvCipher module](AES.srcs/sources_1/new/InvCipher.sv)

![Cipher and InvCipher module](attachments/README/Cipher_and_InvCipher_module.png)

The `Cipher` and `InvCipher` modules perform the AES encryption and decryption operations, respectively.

The following diagram shows the FSM of the `Cipher` module. For more information about the AES encryption operation, refer to chapter 5.1 of the [AES specification](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197-upd1.pdf).

![Cipher FSM](attachments/README/Cipher_FSM.png)

The following diagram shows the FSM of the `InvCipher` module. For more information about the AES decryption operation, refer to chapter 5.3 of the [AES specification](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197-upd1.pdf).

![InvCipher FSM](attachments/README/InvCipher_FSM.png)

### [SubBytes](AES.srcs/sources_1/new/SubBytes.sv) and [InvSubBytes module](AES.srcs/sources_1/new/InvSubBytes.sv)

![SubBytes and InvSubBytes module](attachments/README/SubBytes.png)

The `SubBytes` and `InvSubBytes` modules perform the bytes substitution operation and its inverse, respectively. These modules have a parameter `n` (default value is 16) to determine the bus width (in bytes) of the input and output data.

![Illustration SBox](attachments/README/Illustration_SBox.png)

### [ShiftRows](AES.srcs/sources_1/new/ShiftRows.sv) and [InvShiftRows module](AES.srcs/sources_1/new/InvShiftRows.sv)

![ShiftRows and InvShiftRows module](attachments/README/iState_oState.png)

The `ShiftRows` and `InvShiftRows` modules perform the row shifting operation and its inverse, respectively.

![alt text](attachments/README/Illustration_ShiftRows.png)
![Illustration InvShiftRows](attachments/README/Illustration_InvShiftRows.png)

### [MixColumns](AES.srcs/sources_1/new/MixColumns.sv) and [InvMixColumns module](AES.srcs/sources_1/new/InvMixColumns.sv)

![MixColumns and InvMixColumns module](attachments/README/iState_oState.png)

The `MixColumns` and `InvMixColumns` modules transform the state matrix by multiplying it with a fixed matrix.

![Illustration MixColumns](attachments/README/Illustration_MixColumns.png)

### [AddRoundKey module](AES.srcs/sources_1/new/AddRoundKey.sv)

![AddRoundKey module](attachments/README/AddRoundKey_module.png)

The `AddRoundKey` module performs the XOR operation between the state matrix and the round key.

![Illustration AddRoundKey](attachments/README/Illustration_AddRoundKey.png)

## Synthesis results

Tool: Vivado 2020.2

Device: Arty A7-35T (xc7a35ticsg324-1L)

Resource:

- LUTs: 2225
- FFs: 2068

## References

- [Specification for the ADVANCED ENCRYPTION STANDARD (AES)](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197-upd1.pdf)

<div align="center">

![Visitors](https://api.visitorbadge.io/api/combined?path=https%3A%2F%2Fgithub.com%2FHM-Huong%2Fsystemverilog_AES&countColor=%23b3c6ea&labelStyle=none)

<!-- https://visitorbadge.io/status?path=https://github.com/HM-Huong/systemverilog_AES -->

</div>
