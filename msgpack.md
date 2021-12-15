
## `local mp = require'msgpack'`

MessagePack v5 decoder and encoder for LuaJIT.

---------------------------------------------------- -----------------------------------
`mp.new([mp]) -> mp`                                 create a new mp instance (optional)
__Decoding__
`mp:decode_next(p, [n], [i]) -> next_i, v`           decode value at offset `i` in `p`
`mp:decode_each(p, [n], [i]) -> iter() -> next_i, v` decode all values up to `n` bytes
`mp:encode_buffer([min_size]) -> b`                  create a buffer for encoding
__Encoding__
`b:encode(v)`                                        encode a value (see below)
`b:encode_array(t, [n])`                             encode an array
`b:encode_map(t, [pairs])`                           encode a map
`b:encode_float(x)`                                  encode a float
`b:encode_double(x)`                                 encode a double
`b:encode_bin(v, [n])`                               encode a byte array
`b:encode_ext(type, [n])`                            encode the header for an ext value
`b:encode_ext_int(ctype, x)`                         encode a raw integer (see code)
`b:encode_timestamp(ts)`                             encode a timestamp value
`b:get() -> p, n`                                    get the buffer and its size
`b:tostring() -> s`                                  get the buffer as a string
__Customization__
`mp.nil_key`                                         value to decode nil keys to (skip)
`mp.nan_key`                                         value to decode NaN keys to (skip)
`mp.nil_element`                                     value to decode nil array elements to (`nil`)
`mp.decode_i64`                                      `int64_t` decoder (`tonumber`)
`mp.decode_u64`                                      `uint64_t` decoder (`tonumber`)
`mp.decoder[type] = f(mp, p, i, len) -> next_i, v`   add a decoder for an ext type
`mp:decode_unknown(mp, p, i, len, type_code) end`    decode an unknown ext type
`mp:isarray(t)`                                      decide if `t` is an array or map
`mp.N`                                               key for array element count
`mp.assert(x, err)`                                  custom assert
---------------------------------------------------- -----------------------------------

Decoding behavior:

* `nil` and `NaN` table keys are skipped, unless `mp.nil_key` / `mp.nan_key` is set.
* `nil` array elements create Lua sparse arrays, unless `mp.nil_element` is set.
* array element count is stored in the special `mp.N` key, sparse array or not.
* extension types are decoded with `mp.decoder[type]`, falling back to
`mp:decode_unknown()` (pre-defined as a stub that returns `nil`).
* decoding errors are raised with `mp.assert()` whih defaults to `assert`
(see [errors] for why you'd want to replace this).

Encoding behavior:

* Lua numbers are packed as either integers (the smallest possible) or doubles.
* 64bit cdata numbers are packed as 64bit integers.
* Lua tables are encoded as arrays or maps based on `mp:isarray()` which
by default returns true only if there's a `mp.N` key present in the table
(if `[mp.N] = true` then element count is `#t`).
