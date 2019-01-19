#### Tracing

- Application do not have to deal with generating spans or correlating causality 
- Sidecar generate spans 

Applications need to forward  context headers on outbound calls:

- x-request-id 
- x-b3-traceid 
- x-b3-spanid 
- x-b3-parentspanid 
- x-b3-sampled 
- x-b3-flags 
- x-ot-span-context 



