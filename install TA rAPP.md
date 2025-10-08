# paper note

# TA rAPP Installation log
## Environment
- VMware ubuntu 22.04

## Build TA rApp Docker image
<img width="1825" height="819" alt="image" src="https://github.com/user-attachments/assets/579fe88d-5d16-416c-a9a7-8a62f451fd30" />

## Install TA rApp using Helm






## Upload Test Specification to TA rApp
```
cd nonrtric-rapp-test-automation/Test-Automation-rApp/src/config/
curl -X POST http://192.168.190.128:30082/upload_test_spec \
     -H "Content-Type: application/json" \
     -d @test_spec.json
```
<img width="1421" height="146" alt="image" src="https://github.com/user-attachments/assets/ac16ba6c-f266-4df3-8b1c-f56d90a42152" />



<img width="1748" height="147" alt="image" src="https://github.com/user-attachments/assets/a5599e2f-d0e5-4296-848b-050e0a0b5754" />
