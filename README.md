# packets
A small preprocessor-driven easy-to-use data serialization library

## Defining packets
Inside `packet_def.h`, you can define packets like this:
```c
PACKET(Example,
    // entries here
)
```
The entry types are:
* `INT8`
* `INT16`
* `INT32`
* `INT64`
* `FLOAT`
* `DOUBLE`
* `STRING`

To specify an entry, you can do

`TYPE(name)`

You can also use the `UNSIG` modifier to make the entry unsigned

Example of which is here:
```c
PACKET(Example,
    UNSIG INT16(id)
    INT8(number)
    INT32(bigger_number)
    DOUBLE(decimal_number)
    STRING(some_text)
)
```

The `STRING` type is only able to hold 1024 characters by default (including the null terminator). You can increase this limit by defining the `STR_BUFLEN` with the number of characters before you include `packets.h`

### Arrays

There's also the `ARRAY` type, which takes a list of entries as an argument

```c
PACKET(ArrayExample,
    STRING(name),
    ARRAY(data,
        INT8(integer),
        DOUBLE(decimal)
    )
)
```

## Creating a packet

You can create a new packet like so

```c
Packet_Example* pkt = make_packet(Example);
```

Then you can access its entries as if it was a struct

```c
pkt->bigger_number = 5;
printf("%d\n", pkt->bigger_number); // 5
```

### Initializing arrays

You can initialize arrays to any length using the `make_array` macro

```c
Packet_ArrayExample* array_pkt = make_packet(ArrayExample);
make_array(array_pkt->data, 10); // initialize the array_pkt->data array with the length of 10
```
Then you can access the elements like so
```c
array_pkt->data[2].integer = 3;
```
The `ARRAY` entry type also creates a 2nd entry that holds the length of the array, this should not be modified as it can lead to memory leaks and/or segmentation faults
```c
printf("%d\n", array_pkt->data_arrlen);
```

## Serialization

You can use the `serialized_packet_size` function to create the destination buffer and the `serialize_packet` function to write to the buffer

```c
char serialized_data[serialized_packet_size(pkt)];
serialize_packet(serialized_data);
```

## Deserialization

You can use the `deserialize_unk_packet` to deserialize the packet, or the `deserialize_packet` macro if you know the type

```c
void* deserialized_unk_packet = deserialize_unk_packet(data);
Packet_Example* deserialized_packet = deserialize_packet(Example, data);
```

### Packet types

You can use the `packet_type` macro and then compare the output against the `PacketType` enum to figure out which packet type the pointer points to

```c
int type = packet_type(deserialized_unk_pakcet);
switch (type) {
    case: PacketType_Example: {
        Packet_Example* pkt = (Packet_Example*)deserialized_unk_pakcet;
        // ...
    } break;
    case: PacketType_ArrayExample: {
        Packet_ArrayExample* pkt = (Packet_ArrayExample*)deserialized_unk_pakcet;
        // ...
    } break;
}