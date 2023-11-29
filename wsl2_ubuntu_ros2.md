
Start Ubuntu 22.04 or 20.04 installed from the Microsoft Store and notice the following error being reported upon startup:
```
The attempted operation is not supported for the type of object referenced. Error code: Wsl/Service/0x8007273d Press any key to continue...
```
Solution: \
https://github.com/microsoft/WSL/issues/9331#issuecomment-1356896162
```
netsh winsock reset
```
then restarting PC works.
