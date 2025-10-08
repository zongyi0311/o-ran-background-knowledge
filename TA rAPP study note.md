# paper note
## Backgroung knowledge
- 1. RIC
     - RIC is responsible for intelligent control of the radio access network.
     - It is divided into two components:
          - Near-Real-Time RIC (Near-RT RIC)
          - Non-Real-Time RIC (Non-RT RIC)
     - Near-RT RIC:
          - Handles tasks that require fast response 10ms to 1s.
          - Communicates directly with RAN elements through the E2 interface.
          - Runs xApps, which perform control such as: Handover decisions, Interference mitigation.
     -  Non-RT RIC:
          -  Handle slower or system-level tasks operating >1s like:Policy management and updates, AI/ML training.
          -  Runs rApps that communicate with Near-RT RIC via the A1 interface.
          -  Use the O1 interface to collect data, analyze performance, and assist in decision-making for configuration.
      
- 2. SMO(Service Management and Orchestration)
     - Central component of the O-RAN architecture
     - responsibilities include: managing RAN elements, coordinating applicatios, Orchestrating infrastructure resources
     - standardized Interfaces
       
| Interface | Purpose                                            |
| --------- | -------------------------------------------------- |
| **O1**    | Configuration and monitoring                       |
| **A1**    | Policy exchange between Non-RT RIC and Near-RT RIC |
| **O2**    | Infrastructure management                          |
| **R1**    | Integration with third-party applications          |

     - Intergratio with Non-RT RIC and AI, By leveraging analytics and machine learning, SMO can make smarter operational decisions and adapt dynamically to changing network conditions.

- 
     
     
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
