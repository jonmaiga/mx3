# mx3
Repo with non-cryptographic bit mixer, pseudo random number generator and a hash function. The functions were found semi-algorithmically as detailed in those posts:

* [The construct of a bit mixer](http://jonkagstrom.com/bit-mixer-construction/index.html)
* [Tuning bit mixers](http://jonkagstrom.com/tuning-bit-mixers/index.html)
* [The mx3 mix/prng/hash functions](http://jonkagstrom.com/mx3/index.html)
* [Improved mx3 and the RRC test](http://jonkagstrom.com/mx3/mx3_rev2.html)

In short, first procedurally generated stack machines were used to find good mixer constructions. Then each construction was tuned to see how far it could be pushed and finally I decided to take the simplest most promising mixer and publish the result here.

## Version 2

[Version 2](https://github.com/jonmaiga/mx3/releases/tag/v2.0.0) greatly improves the quality of the mixer by adding an initial xor-shift and adjusting the xor-shift constants. The quality of hash stream mixer is also improved. Article [here](http://jonkagstrom.com/mx3/mx3_rev2.html).

## Version 3

[Version 3](https://github.com/jonmaiga/mx3/releases/tag/v3.0.0) improves mx3::hash with better seeding and speed while maintaining the same good quality.

## mx3::mix

Here is the mixer

    inline uint64_t mix(uint64_t x) {
        x ^= x >> 32;
        x *= 0xbea225f9eb34556d;
        x ^= x >> 29;
        x *= 0xbea225f9eb34556d;
        x ^= x >> 32;
        x *= 0xbea225f9eb34556d;
        x ^= x >> 29;
        return x;
    }

The name 'mx3' comes from this construction (version 1), multiply then xor-shift and repeat three times (referred to as 'mxmxmx' in posts).

## mx3::random

The prng counter based and has a 64-bit state, hence it has a cycle of 2^64. Here is the implementation

    class random {
    public:
        explicit random(uint64_t seed) : _counter(mix(seed + C)) {}
        uint64_t operator()() { return mix(_counter++); }
    private:
        uint64_t _counter;
    };
    

## mx3::hash

Uses the same construction, but to favor speed a lighter version mixes the stream before finalizing the hash with 'mix'.

## Quality

* mx3::mix passes [RRC-64-42-TF2-0.94](https://mostlymangling.blogspot.com/2019/01/better-stronger-mixer-and-test-procedure.html#TestProcedure), also see [mx3 version 2](http://jonkagstrom.com/mx3/mx3_rev2.html)
* mx3::random passes [PractRand](http://pracrand.sourceforge.net/) 2^45 (>32TB) without suspicion
* mx3::hash passes all [SMHasher](https://github.com/rurban/smhasher) tests

## Speed

A short preliminary note on speed (will update if I get around to do better tests)

mixer|MSVC, Threadripper 3970<span>@</span>3.7GHz, ms|MSVC, i7-4790k@4GHz, ms|Clang, i7@2GHz (mac), ms|RRC
--|--|--|--|--
nop|1133|1148|1300|-
splitmix|4047|4716|7917|16
mx3_rev1|4213|4688|7557|21
mx3_rev2|5173|5975|10335|>42

* mx3::hash the speed is according to SMHasher for large keys 8000-9000 MiB/sec and for small keys about 40-50 cycles/hash.

## Feedback

This work is to a large degree empirical without much theoretical attention. It would be interesting to know more of what properties and weaknesses mx3 has, so please don't hesitate to let me know!

## License

The mx3 code is licensed under the [CC0 license](https://creativecommons.org/share-your-work/public-domain/cc0/), but attribution is still appreciated!