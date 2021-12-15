
## `local mp = require'msgpack'`

MessagePack v5 decoder and encoder for LuaJIT.

---------------------------------------------------- -----------------------------------
`mp.new([mp]) -> mp`                                 create a new mp instance (optional)
`mp:decode_next(p, [n], [i]) -> next_i, v`           decode value at offset `i` in `p`
`mp:decode_each(p, [n], [i]) -> iter() -> next_i, v` decode all values up to `n` bytes
`mp:encode_buffer([min_size]) -> b`                  create a buffer for encoding
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
`mp.nil_key`                                         value to decode nil keys to (skip)
`mp.nan_key`                                         value to decode NaN keys to (skip)
`mp.nil_element`                                     value to decode nil array elements to (`nil`)
`mp.decode_i64`                                      `int64_t` decoder (`tonumber`)
`mp.decode_u64`                                      `uint64_t` decoder (`tonumber`)
`mp.decoder[type] = f(mp, p, i, len) -> next_i, v`   add a decoder for an ext type
`mp:decode_unknown(mp, p, i, len, typ) end`          decode an unknown ext type
---------------------------------------------------- -----------------------------------

Decoding behavior:

* `nil` and `NaN` table keys are skipped, unless `mp.nil_key` / `mp.nan_key` is set.
* `nil` array elements create Lua sparse arrays, unless `mp.nil_element` is set.
* extension types are decoded with `mp.decoder[type]`, falling back to
`mp:decode_unknown()` (pre-defined as a stub that returns `nil`).

Encoding behavior:

* Lua numbers are packed as either integers (the smallest possible) or doubles.
* 64bit cdata numbers are packed as 64bit integers.
* Lua tables are packed as arrays (up to `t.n` or `#t`) if `t[1] ~= nil`
or if `next(t) == nil`, otherwise they're packed as maps.
