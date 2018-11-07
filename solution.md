
# Transaction Serialization

We’re going to serialize the TxOut object to a bunch of bytes.

Here is the serialize method for `TxOut`

```python
    def serialize(self):
        '''Returns the byte serialization of the transaction output'''
        # serialize amount, 8 bytes, little endian
        result = int_to_little_endian(self.amount, 8)
        # get the scriptPubkey ready (use self.script_pubkey.serialize())
        raw_script_pubkey = self.script_pubkey.serialize()
        # encode_varint on the length of the scriptPubkey
        result += encode_varint(len(raw_script_pubkey))
        # add the scriptPubKey
        result += raw_script_pubkey
        return result
```
The main thing to note here is that the amount is interpreted as little endian. As explained before, little endian is what Satoshi used in most places, including amount.

Here is the serialize method for `TxIn`:

```python
    def serialize(self):
        '''Returns the byte serialization of the transaction input'''
        # serialize prev_tx, little endian
        result = self.prev_tx[::-1]
        # serialize prev_index, 4 bytes, little endian
        result += int_to_little_endian(self.prev_index, 4)
        # get the scriptSig ready (use self.script_sig.serialize())
        raw_script_sig = self.script_sig.serialize()
        # encode_varint on the length of the scriptSig
        result += encode_varint(len(raw_script_sig))
        # add the scriptSig
        result += raw_script_sig
        # serialize sequence, 4 bytes, little endian
        result += int_to_little_endian(self.sequence, 4)
        return result
```

Once again, the previous transaction, previous index and sequence fields are all in little endian. Previous transaction in particular is tricky as the hexadecimal representation is typically what’s used in block explorers. However, block explorers require the transaction id in big endian, as opposed to what’s specified in the transaction.

### Test Driven Exercise

Now let's write the `serialize()` method for the `Tx` object itself.


```python
import requests
from io import BytesIO
from tx import Tx
from helper import (
    encode_varint,
    int_to_little_endian,
    little_endian_to_int
)

class Tx(Tx):

    def serialize(self):
        '''Returns the byte serialization of the transaction'''
        # serialize version (4 bytes, little endian)
        result = int_to_little_endian(self.version, 4)
        # encode_varint on the number of inputs
        result += encode_varint(len(self.tx_ins))
        # iterate inputs
        for tx_in in self.tx_ins:
            # serialize each input
            result += tx_in.serialize()
        # encode_varint on the number of inputs
        result += encode_varint(len(self.tx_outs))
        # iterate outputs
        for tx_out in self.tx_outs:
            # serialize each output
            result += tx_out.serialize()
        # serialize locktime (4 bytes, little endian)
        result += int_to_little_endian(self.locktime, 4)
        return result
```

One thing that might be interesting to note is that the transaction fee is not specified anywhere! This is because it’s an implied amount. It’s the total of the inputs amounts minus the total of the output amounts.
