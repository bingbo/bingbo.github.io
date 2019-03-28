## CRC32在JS和GOLang中的应用

> js中的实现

```javascript
function crc32_generate(polynomial) {
    var table = new Array()
    var i, j, n
    
    for (i = 0; i < 256; i++) {
        n = i
        for (j = 8; j > 0; j--) {
            if ((n & 1) == 1) {
                n = (n >>> 1) ^ polynomial
            } else {
                n = n >>> 1
            }
        }
        table[i] = n
    }

    return table
}

function crc32_initial() {
    return 0xFFFFFFFF
}

function crc32_final(crc) {
    crc = ~crc
    return crc < 0 ? 0xFFFFFFFF + crc + 1 : crc
}

function crc32_compute_string(polynomial, str) {
    var crc = 0
    var table = crc32_generate(polynomial)
    var i
    
    crc = crc32_initial()
    
    for (i = 0; i < str.length; i++)
        crc = (crc >>> 8) ^ table[str.charCodeAt(i) ^ (crc & 0x000000FF)]
        
    crc = crc32_final(crc)
    return crc
}

function crc32_compute_buffer(polynomial, data) {
    var crc = 0
    var dataView = new DataView(data)
    var table = crc32_generate(polynomial)
    var i
    
    crc = crc32_initial()
    
    for (i = 0; i < dataView.byteLength; i++)
        crc = (crc >>> 8) ^ table[dataView.getUint8(i) ^ (crc & 0x000000FF)]
        
    crc = crc32_final(crc)
    return crc
}

function crc32_reversed(polynomial) {
    reversed = 0
    for (i = 0; i < 32; i++) {
        reversed = reversed << 1
        reversed = reversed | ((polynomial >>> i) & 1)
    }
    return reversed
}

//crc32_compute_string(0x2D02EF8D,'hello')
//3224010476
```


> GoLang的实现

```go
crc32q:=crc32.MakeTable(0x2D02EF8D)
fmt.Printf("%d\n", crc32.Checksum([]byte("hello"), crc32q))
//crc32_compute_string(0x2D02EF8D,'hello')
3224010476
fmt.Println(crc32.ChecksumIEEE([]byte("hello")))
//crc32_compute_string(0x2D02EF8D,'hello')
3224010476
```
