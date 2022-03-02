# Comparing Performance: Loops vs. Iterators

> For a more comprehensive benchmark, you should check using various texts of various sizes as the `contents`, different words and words of different lengths as the `query`, and all kinds of other variations. The point is this: iterators, although a high-level abstraction, get compiled down to roughly the same code as if you’d written the lower-level code yourself. Iterators are one of Rust’s *zero-cost abstractions*, by which we mean using the abstraction imposes no additional runtime overhead. This is analogous to how Bjarne Stroustrup, the original designer and implementor of C++, defines *zero-overhead* in “Foundations of C++” (2012):
> 

### Example 1

[Zero-cost abstractions: performance of for-loop vs. iterators](https://stackoverflow.com/questions/52906921/zero-cost-abstractions-performance-of-for-loop-vs-iterators)

```rust
fn dot_product_loop(x: &[f64], y: &[f64]) -> f64 {
    let mut result: f64 = 0.0;
    for i in 0..min(x.len(), y.len()) {
        result += x[i] * y[i];
    }
    return result;
}

fn dot_product_iter(x: &[f64], y: &[f64]) -> f64 {
    x.iter().zip(y).map(|(&a, &b)| a * b).sum::<f64>()
}

fn dot_product_fold(x: &[f64], y: &[f64]) -> f64 {
    x.iter().zip(y).fold(0f64, |r, (&a, &b)| r + a * b)
}

#[bench]
fn bench_small_loop(b: &mut Bencher) {
    let samples = rand_array(2*LEN as u32);
    b.iter(|| {
        dot_product_loop(&samples[0..LEN], &samples[LEN..2*LEN])
    })
}
#[bench]
fn bench_small_iter(b: &mut Bencher) {
    let samples = rand_array(2 * LEN);
    let s0 = &samples[0..LEN];
    let s1 = &samples[LEN..2 * LEN];
    b.iter(|| dot_product_iter(s0, s1))
}

#[bench]
fn bench_small_fold(b: &mut Bencher) {
    let samples = rand_array(2 * LEN);
    let (s0, s1) = samples.split_at(LEN);
    b.iter(|| dot_product_fold(s0, s1))
}
```

- Output
    
    ```rust
    running 3 tests
    test bench::bench_small_fold ... bench:          18 ns/iter (+/- 1)
    test bench::bench_small_iter ... bench:          21 ns/iter (+/- 1)
    test bench::bench_small_loop ... bench:          24 ns/iter (+/- 1)
    
    test result: ok. 0 passed; 0 failed; 0 ignored; 3 measured; 0 filtered out
    ```
    

## Loop optimization

[](http://fpgacpu.ca/writings/SurveyLoopTransformations.pdf)

[Loop optimization - Wikipedia](https://en.wikipedia.org/wiki/Loop_optimization)

### Fission & Fussion

```rust
int i, a[100], b[100];
for (i = 0; i < 100; i++) {
    a[i] = 1;
    b[i] = 2;
}

```

```rust
int i, a[100], b[100];
for (i = 0; i < 100; i++) {
    a[i] = 1;
}
for (i = 0; i < 100; i++) {
    b[i] = 2;
}
```

### Interchange

```rust
for i from 0 to 10
  for j from 0 to 20
    a[i,j] = i + j
```

```rust
for j from 0 to 20
  for i from 0 to 10
    a[i,j] = i + j
```

### Inversion

```rust
int i, a[100];
i = 0;
while (i < 100) {
  a[i] = 0;
  i++;
}

```

```rust
int i, a[100];
i = 0;
if (i < 100) {
  do {
    a[i] = 0;
    i++;
  } while (i < 100);
}
```

### Reversal

```rust
for (int i=0; i<10; i++) {
	//some work
}
```

```rust
for (int i=9; i--;) {
	//some work
}
```

### Skewing

```fortran
do i = 2, n-1
 do j = 2, m-1
	a[i,j] =
	(a[a-1,j] + a[i,j-1] + a[i+1,j] + a[i,j+1]) / 4 end do
end do
```

```rust
do j = 4, m+n-2
	do i = max(2, j-m+1), min(n-1, j-2)
		a[i,j-i] =
		(a[a-1,j-i] + a[i,j-1-i] + a[i+1,j-i] + a[i,j+1-i]) / 4 end do
end do
```

![Untitled](Comparing%20%2001686/Untitled.png)

### Unrolling (unwinding)

[Loop unrolling - Wikipedia](https://en.wikipedia.org/wiki/Loop_unrolling)

> The goal of loop unwinding is to increase a program's speed by reducing or eliminating instructions that control the loop, such as [pointer arithmetic](https://en.wikipedia.org/wiki/Pointer_arithmetic) and "end of loop" tests on each iteration;[[2]](https://en.wikipedia.org/wiki/Loop_unrolling#cite_note-2) reducing branch penalties; as well as hiding latencies, including the delay in reading data from memory.[[3]](https://en.wikipedia.org/wiki/Loop_unrolling#cite_note-3) To eliminate this [computational overhead](https://en.wikipedia.org/wiki/Computational_overhead), loops can be re-written as a repeated sequence of similar independent statements.[[4]](https://en.wikipedia.org/wiki/Loop_unrolling#cite_note-4)
> 

**Branch Penalty**

[Pipeline stall - Wikipedia](https://en.wikipedia.org/wiki/Pipeline_stall)

[Branch predictor - Wikipedia](https://en.wikipedia.org/wiki/Branch_predictor)

![Untitled](Comparing%20%2001686/Untitled%201.png)

### Vectorization

[Automatic vectorization - Wikipedia](https://en.wikipedia.org/wiki/Automatic_vectorization)

[What is "vectorization"?](https://stackoverflow.com/questions/1422149/what-is-vectorization)

```cpp
for (int i=0; i<16; ++i)
    C[i] = A[i] + B[i];

```

```cpp
for (int i=0; i<16; i+=4) {
    C[i]   = A[i]   + B[i];
    C[i+1] = A[i+1] + B[i+1];
    C[i+2] = A[i+2] + B[i+2];
    C[i+3] = A[i+3] + B[i+3];
}
```

**SIMD**

[Basics of SIMD Programming](http://ftp.cvut.cz/kernel/people/geoff/cell/ps3-linux-docs/CellProgrammingTutorial/BasicsOfSIMDProgramming.html)

![Untitled](Comparing%20%2001686/Untitled%202.png)

### Hoisting

[Loop-invariant code motion - Wikipedia](https://en.wikipedia.org/wiki/Loop-invariant_code_motion)

**End iterator hoisting problem.**

[Pitfalls - 1.48.0](https://www.boost.org/doc/libs/1_48_0/doc/html/foreach/pitfalls.html)

## Example 2

```rust
fn bench_routes(bencher: &mut Bencher) {
		let mut mos = vec![Option::<MovingObject>::None; N];
		let mut routes = vec![Option::<Route>::None; N];
		
		///..skip..		
		let mut loops = 0.0;
		bencher.iter(|| {
		    loops += 1.0;
		    let _r = mos.iter_mut().zip(routes.iter_mut()).for_each(|(mv, rt)| {
		        process_routes(mv, rt, NOW, DELTA * loops);
		    });
		
		    black_box(_r);
		})
}

fn bench_routes_combine(bencher: &mut Bencher) {

    let mut lists = vec![(Option::<MovingObject>::None, Option::<Route>::None); N];
		////..skip..
    let mut loops = 0.0;
    bencher.iter(|| {
        loops += 1.0;
        let _r = lists.iter_mut().for_each(|(mv, rt)| {
            process_routes(mv, rt, NOW, DELTA * loops);
        });
        black_box(_r);
    });
  }
```

- Output
    
    ```rust
    test geo::tests::bench_routes          ... bench:  19,674,157 ns/iter (+/- 2,559,066)
    test geo::tests::bench_routes_combine  ... bench:  22,080,423 ns/iter (+/- 2,384,226)
    test geo::tests::bench_routes_parallel ... bench:   6,489,935 ns/iter (+/- 235,229)
    ```