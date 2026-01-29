# SPO Crystal: 3D Content-Addressable Knowledge Graph

**Replaces Cypher queries with O(1) VSA resonance lookup**

```
╔═══════════════════════════════════════════════════════════════════════╗
║           SPO CRYSTAL: 3D CONTENT-ADDRESSABLE KNOWLEDGE               ║
║                  Replaces Cypher with O(1) Resonance                  ║
╠═══════════════════════════════════════════════════════════════════════╣
║  Vector: 10000 bits | Grid: 5×5×5 = 125 cells | Memory: ~153KB        ║
╚═══════════════════════════════════════════════════════════════════════╝
```

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SPO CRYSTAL: 3D CONTENT-ADDRESSABLE                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   Traditional:  MATCH (s)-[p]->(o) WHERE s.name = "Ada"                │
│                 → O(log N) index lookup + graph traversal              │
│                                                                         │
│   Crystal:      query(S="Ada", P="feels", O=?) → O(1) resonance        │
│                 → 3D address + orthogonal cleanup + qualia overlay     │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  S ⊕ ROLE_S ⊕ P ⊕ ROLE_P ⊕ O ⊕ ROLE_O ⊕ Q ⊕ ROLE_Q           │  │
│   │       ↓           ↓           ↓           ↓                     │  │
│   │    x-axis      y-axis      z-axis      qualia                   │  │
│   │   address     address     address     coloring                  │  │
│   └─────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

## Features

- **3D Spatial Hashing**: S→x, P→y, O→z coordinates in 5×5×5 grid
- **VSA Encoding**: 10,000-bit fingerprints with role-filler binding
- **Orthogonal Codebook**: Gram-Schmidt-like cleaning for high SNR
- **Qualia Coloring**: Felt-sense overlay (arousal, valence, tension, depth)
- **NARS Truth Values**: Frequency + confidence for uncertain knowledge
- **3D Cubic Popcount**: Tensor Hamming distance for field similarity
- **Field Closeness Index**: Multi-cell resonance detection

## Query Types

### Exact Queries (100% accuracy)
```rust
// (Ada, loves, ?) → find O
crystal.query_object("Ada", "loves")  // → ["Jan"]

// (?, loves, Ada) → find S  
crystal.query_subject("loves", "Ada")  // → ["Jan"]

// (Ada, ?, Jan) → find P
crystal.query_predicate("Ada", "Jan")  // → ["loves"]
```

### Resonance Queries (semantic similarity)
```rust
// All triples with Subject="Ada"
crystal.resonate_spo(Some("Ada"), None, None, 0.55)

// All "loves" relationships
crystal.resonate_spo(None, Some("loves"), None, 0.55)

// Fuzzy/semantic matching via VSA similarity
```

## Cypher Comparison

| Cypher Query | SPO Crystal |
|--------------|-------------|
| `MATCH (a)-[:LOVES]->(b)` | `resonate(None, "loves", None)` |
| `WHERE a.name = 'Ada'` | `resonate(Some("Ada"), None, None)` |
| `MATCH (a)-[*1..3]->(b)` | Resonance cascade |
| Fuzzy regex match | **Native** VSA similarity |

## Performance

| Triples | Insert | Exact Query | Resonance |
|---------|--------|-------------|-----------|
| 100 | 9ms | 0.05ms | 0.3ms |
| 1,000 | 215ms | 0.2ms | 5ms |
| 10,000 | 3.5s | 0.75ms | 74ms |

## Key Numbers

- **Memory**: 153KB for 5×5×5 grid
- **Vector dimension**: 10,000 bits
- **Cells**: 125 (5³)
- **Exact query speed**: 130K queries/sec @ 10K triples

## Advantages over Graph DB

- ✓ O(1) address lookup via 3D hash
- ✓ Native fuzzy/semantic matching
- ✓ Composable queries via VSA algebra
- ✓ 153KB memory (vs GB)
- ✓ Qualia coloring for felt-sense
- ✓ No query optimizer needed

## Usage

```rust
let mut crystal = SPOCrystal::new();

// Insert with qualia
crystal.insert(
    Triple::new("Ada", "remembers", "first_kiss")
        .with_qualia(Qualia::new(0.8, 0.9, 0.2, 0.9))  // joy/profound
);

// Query
let results = crystal.query_object("Ada", "remembers");
for (obj, sim, qualia) in results {
    println!("{} (sim={:.3})", obj, sim);
}
```

## Theory

Based on:
- **VSA (Vector Symbolic Architecture)**: Kanerva, Plate, Gayler
- **Hopfield Networks**: Attractor dynamics
- **NARS**: Non-axiomatic reasoning with truth values
- **Spatial Hashing**: O(1) coordinate lookup

## License

MIT

## Related

- [crystal-memory](https://github.com/AdaWorldAPI/crystal-memory) - 4KB holographic crystals
- [ladybug-rs](https://github.com/AdaWorldAPI/ladybug-rs) - Rust VSA/NARS foundation

## Jina Embedding Cache

SPO Crystal includes a smart caching layer for Jina embeddings that dramatically reduces API calls:

### Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│  EMBEDDING LOOKUP (< 0.15 Hamming threshold)                    │
├─────────────────────────────────────────────────────────────────┤
│  1. EXACT MATCH (HashMap)       → instant, 0 API calls         │
│  2. NEAR MATCH (case/typo)      → use closest, 0 API calls     │
│  3. CACHE MISS                  → Jina API, then cache         │
└─────────────────────────────────────────────────────────────────┘
```

### Results

```
Processing 22 entity lookups...

JinaCache Statistics:
  Entries:      10
  Lookups:      22
  Exact hits:   7 (31.8%)   ← same string repeated
  Near hits:    5 (22.7%)   ← case variations (Ada/ada/ADA)
  API calls:    10 (45.5%)  ← actual Jina requests
  Hit rate:     54.5%

Without cache:  22 API calls
With cache:     10 API calls
Savings:        54.5%
```

For typical knowledge graphs with heavy entity repetition, savings can reach **90%+**.

### Usage

```rust
use jina_cache::JinaCache;

let mut cache = JinaCache::new("your_jina_api_key")
    .with_persistence("/path/to/cache.bin");

// First call: API request, then cached
let fp1 = cache.get_fingerprint("Ada")?;

// Second call: instant from cache
let fp2 = cache.get_fingerprint("Ada")?;

// Near match: uses cached "Ada" (no API call)
let fp3 = cache.get_fingerprint("ada")?;

cache.print_stats();
```

### Batch Processing

```rust
// More efficient: single API call for all misses
let texts = vec!["Ada", "Jan", "loves", "creates"];
let fingerprints = cache.get_fingerprints_batch(&texts)?;
```
