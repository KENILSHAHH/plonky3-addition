# Proving addition of two numbers is equal to a third number

This project demonstrates how to use [Plonky3](https://github.com/Ponky3/plonky3), a fast, Rust-based STARK proof system, to generate a zero-knowledge proof for the addition of two numbers. This example showcases how to set up an **execution trace**, define the **AIR (Algebraic Intermediate Representation)**, and create a proof that can be verified to confirm the addition result.

## Project Overview

The main goal is to prove that the sum of two given numbers, `a` and `b`, is equal to an expected result `c`. This is achieved using a basic **AdditionAir** configuration in Plonky3, which ensures that the constraints of addition are correctly defined and verified.

## Prerequisites

- [Rust](https://www.rust-lang.org/tools/install) installed
- Plonky3 crate dependencies added to `Cargo.toml`
- Basic understanding of zero-knowledge proofs and STARKs

## Code Structure

- **AdditionAir**: Defines the AIR with constraints to ensure `a + b = c`.
- **Execution Trace**: Contains the input values (`a`, `b`, and `c`) and outputs for verification.
- **Prover and Verifier**: Uses Plonky3's `prove` and `verify` functions to prove and verify the computation.

## Quick Start
Start running the program which will eventually create a proof and verify it 

``` bash
git clone https://github.com/kenilshahh/plonky3-addition
cargo run
```

## Getting Started

### Step 1: Define the Addition AIR

The **AdditionAir** structure represents the constraints for addition and enforces the rule that `a + b = c`.

```rust
pub struct AdditionAir {
    pub a: u32,
    pub b: u32,
    pub c: u32,
}

impl<F: Field> BaseAir<F> for AdditionAir {
    fn width(&self) -> usize {
        3
    }
}

impl<AB: AirBuilder> Air<AB> for AdditionAir {
    fn eval(&self, builder: &mut AB) {
        let main = builder.main();
        let local = main.row_slice(0);

        builder
            .when_first_row()
            .assert_eq(local[0] + local[1], local[2]);

        let final_value = AB::Expr::from_canonical_u32(self.c);
        builder.when_first_row().assert_eq(local[2], final_value);
    }
}
```


The `eval` function in `AdditionAir` enforces that `local[0] + local[1] == local[2]` (where `local[0]` is `a`, `local[1]` is `b`, and `local[2]` is `c`). This constraint ensures that the AIR (Algebraic Intermediate Representation) correctly represents the addition rule for any given inputs `a` and `b`, validating that their sum is equal to `c`.

## Step 2: Generating the Execution Trace

The **execution trace** is a matrix of values that represents the computation at each step. In this example, the trace matrix has 3 columns, one for each of `a`, `b`, and `c`, and contains rows corresponding to the number of steps in the proof. The `generate_addition_trace` function populates this matrix with values for `a`, `b`, and `c` in the first row, while the remaining rows are initialized to zero.

```rust
pub fn generate_addition_trace<F: Field>(a: u32, b: u32, c: u32) -> RowMajorMatrix<F> {
    let mut values = Vec::with_capacity(3 * 4);
    let a_f = F::from_canonical_u32(a);
    let b_f = F::from_canonical_u32(b);
    let c_f = F::from_canonical_u32(c);

    values.push(a_f);
    values.push(b_f);
    values.push(c_f);

    // Initializing the remaining rows with zeros
    for _ in 1..4 {
        values.push(F::zero());
        values.push(F::zero());
        values.push(F::zero());
    }

    RowMajorMatrix::new(values, 3)
}
```
The generate_addition_trace function returns a RowMajorMatrix populated with the values for a, b, and c, ensuring that the addition of a and b results in c in the trace matrix.

## Step 3: Proving and Verifying the Addition

To prove and verify that a + b = c using Plonky3, we need to set up a configuration with a prover and verifier. In main, the configuration includes the necessary elements to execute and validate the proof, such as the FRI (Fast Reed-Solomon IOP) and Merkle-based commitment schemes for secure proof construction.

 ``` rust
 fn main() -> Result<(), impl Debug> {
    let a = 4;
    let b = 5;
    let c = 9;

    let air = AdditionAir { a, b, c };
    let trace = generate_addition_trace::<Val>(a, b, c);

    let proof = prove(&config, &air, &mut challenger, trace, &vec![]);

    verify(&config, &air, &mut challenger, &proof, &vec![])
}
```

