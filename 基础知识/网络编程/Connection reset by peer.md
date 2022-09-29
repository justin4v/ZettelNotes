#Problem-and-Solutions #Problems

# 原因
- The remote server has sent you a **RST packet**, which indicates an immediate dropping of the connection, rather than the usual handshake. 
- This bypasses the normal **half-closed state transition**。
- It cannot be called 'connection reset by server' because it **can be sent by the client or the server**.