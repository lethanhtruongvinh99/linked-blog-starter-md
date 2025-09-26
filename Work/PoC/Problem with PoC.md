about the file format, will it be considered by AMS Authority, for example, FF for CA, JSON for US. 
Who will be in charge for the transform data
Who will add the header 
The mask for FF file (the first row): \$\$\$MSGSTART: + Sender ID[20] + Receiver ID[20] + Message Type[10] + FlatFile Ref No.[15]

The mask for FF Ref No: ScrumId[4] + prefixId[3] + Sysdate(YYMMDD) + sequence(10digits)
```
function extractMessageFields(message) {
	// remove the slash in js because the md dont support dollar sign
	const prefix = "$$$MSGSTART:";
	const senderLength = 20;
	  const receiverLength = 20;
	  const messageTypeLength = 10;
	  const flatFileRefLength = 15;

  // Check if message starts with the expected prefix
  if (!message.startsWith(prefix)) {
    throw new Error("Invalid message format: missing prefix.");
  }

  // Remove prefix and trim any extra whitespace
  const payload = message.slice(prefix.length).trim();

  // Validate payload length
  const expectedLength = senderLength + receiverLength + messageTypeLength + flatFileRefLength;
  if (payload.length < expectedLength) {
    throw new Error("Message is too short for the expected mask.");
  }

  // Extract fields
  const senderID = payload.slice(0, senderLength).trim();
  const receiverID = payload.slice(senderLength, senderLength + receiverLength).trim();
  const messageType = payload.slice(senderLength + receiverLength, senderLength + receiverLength + messageTypeLength).trim();
  const flatFileRef = payload.slice(senderLength + receiverLength + messageTypeLength, expectedLength).trim();

  return {
    senderID,
    receiverID,
    messageType,
    flatFileRef
  };
}
```


```
function buildMessage(senderID, receiverID, messageType, flatFileRef) {
  const prefix = "$$$MSGSTART:";

  const sender = senderID.toString().padEnd(20, ' ');
  const receiver = receiverID.toString().padEnd(20, ' ');
  const type = messageType.toString().padEnd(10, ' ');
  const ref = flatFileRef.toString().padEnd(15, ' ');

  return `${prefix}${sender}${receiver}${type}${ref}`;
}
```

```
function generateFlatFileRef(scrumId, prefixId, sequenceNumber) {
  // Ensure inputs are strings and properly padded
  const paddedScrumId = scrumId.toString().padEnd(4, '0').slice(0, 4);
  const paddedPrefixId = prefixId.toString().padEnd(3, '0').slice(0, 3);
  const paddedSequence = sequenceNumber.toString().padStart(10, '0').slice(-10);

  // Get current date in YYMMDD format
  const now = new Date();
  const year = now.getFullYear().toString().slice(-2);
  const month = String(now.getMonth() + 1).padStart(2, '0');
  const day = String(now.getDate()).padStart(2, '0');
  const sysDate = `${year}${month}${day}`;

  // Build the FlatFile Ref No
  return `${paddedScrumId}${paddedPrefixId}${sysDate}${paddedSequence}`;
}
```

