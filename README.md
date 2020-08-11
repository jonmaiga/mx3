# mx3
Repo with non-cryptographic bit mixer, pseudo random number generator and a hash function. The functions were found semi-algorithmically as detailed in those posts:

* [The construct of a bit mixer](http://jonkagstrom.com/bit-mixer-construction/index.html)
* [Tuning bit mixers](http://jonkagstrom.com/tuning-bit-mixers/index.html)
* [The mx3 mix/prng/hash functions](http://jonkagstrom.com/mx3/index.html)

In short, first procedurally generated stack machines were used to find good mixer constructions. Then each construction was tuned to see how far it could be pushed and finally I decided to take the simplest most promising mixer and publish the result here.

## mx3::mix

Here is the mixer

    inline uint64_t mix(uint64_t x) {
        x *= 0xbea225f9eb34556d;
        x ^= x >> 33;
        x *= 0xbea225f9eb34556d;
        x ^= x >> 29;
        x *= 0xbea225f9eb34556d;
        x ^= x >> 39;
        return x;
    }

The name 'mx3' comes from this construction, multiply then xor-shift and repeat three times (referred to as 'mxmxmx' in posts).

## mx3::random

The prng counter based and has a 64-bit state, hence it has a cycle of 2^64. Here is the implementation

    class random {
    public:
        explicit random(uint64_t seed) : _counter(seed) {}
        uint64_t operator()() { return mix(_counter++); }
    private:
        uint64_t _counter;
    };
    

## mx3::hash

Uses the same construction, but to favor speed a lighter version mixes the stream before finalizing the hash with 'mix'.

## Quality

* mx3::random passes [PractRand](http://pracrand.sourceforge.net/) 2^45 (>32TB) without suspicion
* mx3::hash passes all [SMHasher](https://github.com/rurban/smhasher) tests

## Speed

A short preliminary note on speed (will update if I get around to do better tests)

* mx3::mix is in my limited tests as fast as SplitMix (even though it has one more multiplication!).

* mx3::random is as reference again close to SplitMix.

* mx3::hash the speed is according to SMHasher for large keys 8000-9000 MiB/sec and for small keys about 40-50 cycles/hash.

## Feedback

This work is to a large degree empirical without much theoretical attention. It would be interesting to know more of what properties and weaknesses mx3 has, so please don't hesitate to let me know!

## License

The mx3 code is licensed under the [CC0 license](https://creativecommons.org/share-your-work/public-domain/cc0/), but attribution is still appreciated!