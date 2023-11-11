# DevSecOps Terms

## Agile SDLC - Rapid development, Rapid deployment
- Method of software development that is characterized by
  - Division of tasks into short phases of work - **sprints of 2-3 weeks**
  - Frequent reassessment - testing: functional, security, user
  - Adaptation of plans 
- [An Overview of Agile Development by ForrestKnight](https://www.youtube.com/watch?v=QLvBK9stdoM)
- [Agile Software Development by MIT Courseware](https://www.youtube.com/watch?v=UxMpn92vGXs)
  

## DevOps
- Devops is a set of practices that increases the speed of **dev**eloping high quality software and **op**eration services.
- Derived from Agile methodology.
- This is done through automation, collaboration, fast feedback, and iterative improvement.
- [What is DevOps? by GitLab](https://about.gitlab.com/topics/devops/)
- [DevOps CI/CD Explained in 100 Seconds by Fireship](https://www.youtube.com/watch?v=scEDHsr3APg)

## CI/CD
- Continuous Integration - Team members integrate their work in a **shared repository**. Git commits. **Basic testing**. 
- Continuous Delivery - Deploying on a **pre-production or staging environment** and performing **automated tests** to ensure the build is ready for production/release.
- Continuous Deployment (not common) - **Automatically deploy the build** to production.
- [What is CI/CD Pipeline? by School of Basics](https://www.youtube.com/watch?v=k2aNsQKwyOo) **(See image in timestamp 8:40)**

## Shift Left - Fail soon to fix soon.
- In traditional SDLC, the stages include `Plan > Code > Build > Test > Deploy > Monitor`, where Security comes during the 'Test' or 'Deploy' phase.
- Shift left means **conducting security testing sooner and more often** in the software development phase, i.e. starting from the 'Code' phase and then in the 'Test' and 'Deploy' phases.
- Security testing starts from the source - SAST, DAST, SCA
- [What is Shift Left Security by Fortinet](https://www.fortinet.com/fr/resources/cyberglossary/shift-left-security)
- [Shift Left Security: Meaning and Real World Usage by CoderDave](https://www.youtube.com/watch?v=94AIX5oArIw)

## SAST, DAST, SCA
- [SCA vs SAST by Tanya Janca](https://www.youtube.com/watch?v=Q2rE5XIO1Ug)
- [SAST vs DAST by Synopsys](https://www.synopsys.com/blogs/software-security/sast-vs-dast-difference.html)

### SAST - Static Application Security Testing
- Analyze the source code (written by the dev team) of the product for vulnerabilities.
- Manual or automated **white box testing**.
- Automated tests may have false positives. Time consuming
- Happens earlier in the SDLC.

### DAST - Dynamic Application Security Testing
- Analyze a running build of the product for vulnerabilities.
- Manual or automated **black box testing**.
- Automated tests may have false positives. Time consuming
- Happens later in the SDLC.

### SCA - Software Composition Analysis
- Analyze the **security of the dependencies** used in a software project.
- Checks for known/disclosed vulnerabilities in these dependencies.
- More accurate. SCA scans run faster.
- Ideally should happen earlier in the SDLC. 
