# Symbol/Zebra MC18/PS20 cradle protocol

## Packets

### Tx

       /  / Literal: 1
       |  /  / Command
       |  |  /  / Packet length
       |  |  |  /     / Data
       |  |  |  |     /  / Literal: 3
       |  |  |  |     |  /  / Checksum: XOR each byte
       |--|--|--|-----|--|--/  / Literal: 0 (not in length)
    Tx: 01|65|07|nn nn|03|67|00


### Rx

       /  / Literal: 1
       |  /  / Packet length
       |  |  /  / Literal: 7
       |  |  |  /     / Data
       |  |  |  |     /  / Literal: 3
       |  |  |  |     |  /  / Checksum: XOR each byte
    Rx: 01|07|07|nn nn|03|9b

## Commands

* `0x65`: Release
* `0x66`: Read part number
* `0x69`: Read serial number
* `0x6b`: Read manufacture date
* `0x6d`: Read hardware ID
* `0x6f`: Read row
* `0x70`: Read col
* `0x71`: Read state
* `0x72`: Firmware update?
* `0x73`: Set state
* `0x74`: Set row
* `0x75`: Set col
* `0x76`: Blink
* `0x77`: Read firmware version
* `0x7a`: Get voltage 1
* `0x7b`: Get voltage 2
* `0x7c`: Get current
* `0x7d`: Set wall
* `0x7e`: Read wall
* `0x81`: Turn off

## Latch

## Release

                /  / Interval
                |  /     / LED on time
                |  |     /     / LED off time
                |  |     |     /  / Smooth
                |--|-----|-----|--|
    Tx: 01 65 0b|0b|01 f4|01 f4|00|03 67 00         11s LED
    Tx: 01 65 0b|0a|01 f4|01 f4|00|03 66 00         10s LED
    Tx: 01 65 0b|0a|00 00|00 00|00|03 66 00         10s no LED
    Tx: 01 65 0b|1e|01 f4|01 f4|00|03 72 00         30s LED
    Tx: 01 65 0b|1e|01 f5|01 f4|01|03 72 00         30s LED smooth
    Rx: 01 05 07|03 00   # Ack


## LEDs

### Blink

                /     / LED on time
                |     /     / LED off time
                |     |     /     / Count
                |     |     |     /  / Smooth
                |-----|-----|-----|--|
    Tx: 01 76 0c|01 f4|01 f4|00 05|00|03 7d 00    # 500 500 x 5
    Tx: 01 76 0c|01 f4|01 f4|00 63|00|03 1b 00    # 500 500 x 99
    Tx: 01 76 0c|00 c8|00 c8|00 02|00|03 7a 00    # 2 x 200ms, 200ms
    Rx: 01 05 07|03 00   # Ack

## Turn off

    Tx: 01 81 05|03 86 00
    Rx: 01 05 07|03 00   # Ack


## Cradle state


                /  / State
    Tx: 01 73 06|31|03 46 00
    Rx: 01 05 07|03 00   # Ack

                /  / State
    Tx: 01 71 05|03 76 00
    Rx: 01 06 07|30|03 33

State bits

        /// 000
           //// 1111
               / Fast charge
    msb nnnnnnnn lsb

## Cradle location

### Row

    Tx: 01 6f 05|03 68 00      # read row
                 /   / Row 65535
    Rx: 01 07 07|ff ff|03 02  # row response

                 /   / Row 65535
    Tx: 01 74 07|ff ff|03 71 00 # Set row
    Rx: 01 05 07|03 00   # Ack

### Column

    Tx: 01 70 05|03 77 00       # read col
                 /   / Row 65534
    Rx: 01 07 07|ff fe|03 03    # col response

                 /   / Row 65534
    Tx: 01 75 07|ff fe|03 71 00 # Set Col
    Rx: 01 05 07|03 00   # Ack

### Wall

    Tx: 01 7e 05|03 79 00       # read wall
                 /   / Wall 65533
    Rx: 01 07 07|ff fd|03 00    # col response

                 /   / Wall 65533
    Tx: 01 7d 07|ff fd|03 7a 00 # Set wall
    Rx: 01 05 07|03 00   # Ack


## Cradle power

### Voltage

FIXME update

    5.2052784 / 5.21261
    Tx: 01 7a 05|03 7d 00
    Rx: 01 07 07|02 c6|03 c6
    
    0000 0010 1100 0110
    seee eeff ffff ffff

    3.2600195
    Tx: 01 7b 05|03 7c 00
    Rx: 01 07 07|02 9b|03 9b
    
    0000001010011011


### Solenoid current

    Tx: 01 7c 05|03 7b 00
                 /   / Current: 0.0mA
    Rx: 01 07 07|00 00|03 02

### Fast charge

See Cradle State.


## Cradle info

### Serial number
    Tx: 01 69 05|03 6e 00
                 /                                          / Serial: 210355230239620
    Rx: 01 14 07|32 31 30 33 35 35 32 33 30 32 33 39 36 32 30|03 2c

### Hardware ID
    Tx: 01 6d 05|03 6a 00
                 // ID: 1
    Rx: 01 06 07|01|03 02

### Part number
    Tx: 01 66 05|03 61 00
                 /                                                   / Part: CRD-MC18-1SLOT-01
    Rx: 01 17 07|43 52 44 2d 4d 43 31 38 2d 31 53 4c 4f 54 2d 30 31 00|03 59

### Manufacture date
    Tx: 01 6b 05|03 6c 00
                 /      / Date: 4.2.21
    Rx: 01 08 07|04 02 15|03 1e

### Firmware version
    Tx: 01 77 05|03 70 00
                 /   / version: 5.1
    Rx: 01 07 07|05 01|03 06

