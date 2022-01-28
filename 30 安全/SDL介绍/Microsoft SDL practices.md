#  What are the Microsoft SDL practices?

The Security Development Lifecycle (SDL) consists of a set of practices that support security assurance and compliance requirements. The SDL helps developers build more secure software by reducing the number and severity of vulnerabilities in software, while reducing development cost. 

![No Data Available](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RE2Ih21?ver=f9fb&q=90&m=6&h=201&w=358&b=%23FFFFFFFF&l=f&o=t&aim=true)

### Provide Training

Ensure everyone understands security best practices.

[Learn more ](https://www.microsoft.com/en-us/securityengineering/sdl/practices#practice1)

![No Data Available](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RE2Itfi?ver=6210&q=90&m=6&h=201&w=358&b=%23FFFFFFFF&l=f&o=t&aim=true)

### Define Security Requirements

Continually update security requirements to reflect changes in functionality and to the regulatory and threat landscape.

[Learn more ](https://www.microsoft.com/en-us/securityengineering/sdl/practices#practice2)

![No Data Available](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RE2IqPr?ver=84ee&q=90&m=6&h=201&w=358&b=%23FFFFFFFF&l=f&o=t&aim=true)

### Define Metrics and Compliance Reporting

Identify the minimum acceptable levels of security quality and how engineering teams will be held accountable.

[Learn more ](https://www.microsoft.com/en-us/securityengineering/sdl/practices#practice3)

![No Data Available](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RE2IoQK?ver=8629&q=90&m=6&h=201&w=358&b=%23FFFFFFFF&l=f&o=t&aim=true)

### Perform Threat Modeling

Use threat modeling to identify security vulnerabilities, determine risk, and identify mitigations.

[Learn more ](https://www.microsoft.com/en-us/securityengineering/sdl/practices#practice4)

![No Data Available](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RE2IAFC?ver=3c40&q=90&m=6&h=201&w=358&b=%23FFFFFFFF&l=f&o=t&aim=true)

### Establish Design Requirements

Define standard security features that all engineers should use.

[Learn more ](https://www.microsoft.com/en-us/securityengineering/sdl/practices#practice5)

![No Data Available](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RE2IzrL?ver=f1b0&q=90&m=6&h=201&w=358&b=%23FFFFFFFF&l=f&o=t&aim=true)

### Define and Use Cryptography Standards

Ensure the right cryptographic solutions are used to protect data.

[Learn more ](https://www.microsoft.com/en-us/securityengineering/sdl/practices#practice6)

![No Data Available](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RE2Itfl?ver=f331&q=90&m=6&h=201&w=358&b=%23FFFFFFFF&l=f&o=t&aim=true)

### Manage the Security Risk of Using Third-Party Components

Keep an inventory of third-party components and create a plan to evaluate reported vulnerabilities.

[Learn more ](https://www.microsoft.com/en-us/securityengineering/sdl/practices#practice7)

![No Data Available](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RE2I7t3?ver=3ac1&q=90&m=6&h=201&w=358&b=%23FFFFFFFF&l=f&o=t&aim=true)

### Use Approved Tools

Define and publish a list of approved tools and their associated security checks.

[Learn more ](https://www.microsoft.com/en-us/securityengineering/sdl/practices#practice8)

![No Data Available](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RE2IzrO?ver=510f&q=90&m=6&h=201&w=358&b=%23FFFFFFFF&l=f&o=t&aim=true)

### Perform Static Analysis Security Testing (SAST)

Analyze source code before compiling to validate the use of secure coding policies.

[Learn more ](https://www.microsoft.com/en-us/securityengineering/sdl/practices#practice9)

![No Data Available](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RE2IoQN?ver=2b3f&q=90&m=6&h=201&w=358&b=%23FFFFFFFF&l=f&o=t&aim=true)

### Perform Dynamic Analysis Security Testing (DAST)

Perform run-time verification of fully compiled software to test security of fully integrated and running code.

[Learn more ](https://www.microsoft.com/en-us/securityengineering/sdl/practices#practice10)

![No Data Available](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RE2I7t4?ver=031f&q=90&m=6&h=201&w=358&b=%23FFFFFFFF&l=f&o=t&aim=true)

### Perform Penetration Testing

Uncover potential vulnerabilities resulting from coding errors, system configuration faults, or other operational deployment weaknesses.

[Learn more ](https://www.microsoft.com/en-us/securityengineering/sdl/practices#practice11)

![No Data Available](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RE2IAFF?ver=5eff&q=90&m=6&h=201&w=358&b=%23FFFFFFFF&l=f&o=t&aim=true)

### Establish a Standard Incident Response Process

Prepare an Incident Response Plan to address new threats that can emerge over time.

[Learn more ](https://www.microsoft.com/en-us/securityengineering/sdl/practices#practice12)



### Practice #1 - Provide Training

Security is everyone’s job. Developers, service engineers, and program and product managers must understand security basics and know how to build security into software and services to make products more secure while still addressing business needs and delivering user value.

Effective training will complement and re-enforce security policies, SDL practices, standards, and requirements of software security, and be guided by insights derived through data or newly available technical capabilities.

Although security is everyone’s job, it’s important to remember that not everyone needs to be a security expert nor strive to become a proficient penetration tester. However, ensuring everyone understands the attacker’s perspective, their goals, and the art of the possible will help capture the attention of everyone and raise the collective knowledge bar.

 

### Practice #2 - Define Security Requirements

The need to consider security and privacy is a fundamental aspect of developing highly secure applications and systems and regardless of development methodology being used, security requirements must be continually updated to reflect changes in required functionality and changes to the threat landscape. Obviously, the optimal time to define the security requirements is during the initial design and planning stages as this allows development teams to integrate security in ways that minimize disruption. Factors that influence security requirements include (but are not limited to) the legal and industry requirements, internal standards and coding practices, review of previous incidents, and known threats. These requirements should be tracked through either a work-tracking system or through telemetry derived from the engineering pipeline.

 

**Useful links**:

[Azure DevOps / Azure Boards](https://azure.microsoft.com/en-us/services/devops/boards/)

 

### Practice #3 - Define Metrics and Compliance Reporting 

It is essential to define the minimum acceptable levels of security quality and to hold engineering teams accountable to meeting that criteria. Defining these early helps a team understand risks associated with security issues, identify and fix security defects during development, and apply the standards throughout the entire project. Setting a meaningful bug bar involves clearly defining the severity thresholds of security vulnerabilities (for example, all known vulnerabilities discovered with a “critical” or “important” severity rating must be fixed with a specified time frame) and never relaxing it once it's been set.

In order to track key performance indicators (KPIs) and ensure security tasks are completed, the bug tracking and/or work tracking mechanisms used by an organization (such as Azure DevOps) should allow for security defects and security work items to be clearly labeled as security and marked with their appropriate security severity. This allows for accurate tracking and reporting of security work.

 

**Useful Links**:

[SDL Privacy Bug Bar Sample](https://msdn.microsoft.com/en-us/library/cc307403.aspx)

[Add or modify a field to track data requirements in Azure Devops](https://docs.microsoft.com/en-us/azure/devops/reference/add-modify-field?view=tfs-2018&viewFallbackFrom=vsts)

[Add a Security Bug Bar to Microsoft Team Foundation Server 2010](https://msdn.microsoft.com/en-us/magazine/ee336031.aspx)(legacy resource)

[SDL Security Bug Bar Sample](https://msdn.microsoft.com/library/cc307404.aspx)

 

### Practice #4 - Perform Threat Modeling

Threat modeling should be used in environments where there is meaningful security risk. Threat modeling can be applied at the component, application, or system level. It is a practice that allows development teams to consider, document, and (importantly) discuss the security implications of designs in the context of their planned operational environment and in a structured fashion.

Applying a structured approach to threat scenarios helps a team more effectively and less expensively identify security vulnerabilities, determine risks from those threats, and then make security feature selections and establish appropriate mitigations.

 

**Useful Links**:

[Threat Modeling ](https://www.microsoft.com/en-us/securityengineering/sdl/threatmodeling)

 

### Practice #5 - Establish Design Requirements

The SDL is typically thought of as assurance activities that help engineers implement “secure features”, in that the features are well engineered with respect to security. To achieve this, engineers will typically rely on security features, such as cryptography, authentication, logging, and others. In many cases, the selection or implementation of security features has proven to be so complicated that design or implementation choices are likely to result in vulnerabilities. Therefore, it’s crucially important that these are applied consistently and with a consistent understanding of the protection they provide. 

 

### Practice #6 - Define and Use Cryptography Standards

With the rise of mobile and cloud computing, it’s critically important to ensure all data, including security-sensitive information and management and control data, is protected from unintended disclosure or alteration when it’s being transmitted or stored. Encryption is typically used to achieve this. Making an incorrect choice in the use of any aspect of cryptography can be catastrophic, and it’s best to develop clear encryption standards that provide specifics on every element of the encryption implementation. This should be left to experts. A good general rule is to only use industry-vetted encryption libraries and ensure they’re implemented in a way that allows them to be easily replaced if needed.

 

**Useful Links**:

[Microsoft SDL Cryptographic Recommendations](http://download.microsoft.com/download/6/3/A/63AFA3DF-BB84-4B38-8704-B27605B99DA7/Microsoft SDL Cryptographic Recommendations.pdf)

 

### Practice #7 - Manage the Security Risk of Using Third-Party Components

Today, the vast majority of software projects are built using third-party components (both commercial and open source). When selecting third-party components to use, it’s important to understand the impact that a security vulnerability in them could have to the security of the larger system into which they are integrated. Having an accurate inventory of third-party components and a plan to respond when new vulnerabilities are discovered will go a long way toward mitigating this risk, but additional validation should be considered, depending on your organization's risk appetite, the type of component used, and potential impact of a security vulnerability. [Learn more about managing security risks of using third-party components such as open source software.](https://www.microsoft.com/en-us/securityengineering/opensource/)

 

**Useful Links**:

[Managing Security Risks Inherent in the Use of Third-Party Components](https://safecode.org/wp-content/uploads/2017/05/SAFECode_TPC_Whitepaper.pdf)

[Managing Security Risks Inherent in the Use of Open-Source Software](https://www.microsoft.com/en-us/securityengineering/opensource/)

 

### Practice #8 - Use Approved Tools

Define and publish a list of approved tools and their associated security checks, such as compiler/linker options and warnings. Engineers should strive to use the latest version of approved tools, such as compiler versions, and to take advantage of new security analysis functionality and protections.

 

**Useful Links**:

[Recommended Tools, Compilers and Options for x86, x64 and ARM](http://download.microsoft.com/download/6/3/A/63AFA3DF-BB84-4B38-8704-B27605B99DA7/Recommended Tools, Compilers and Options for x86, x64 and ARM.pdf)(legacy tools)

[SDL Tools](https://www.microsoft.com/en-us/securityengineering/sdl/resources)

 

### Practice #9 - Perform Static Analysis Security Testing (SAST)

Analyzing the source code prior to compilation provides a highly scalable method of security code review and helps ensure that secure coding policies are being followed. SAST is typically integrated into the commit pipeline to identify vulnerabilities each time the software is built or packaged. However, some offerings integrate into the developer environment to spot certain flaws such as the existence of unsafe or other banned functions and replace those with safer alternatives as the developer is actively coding. There is no one size fits all solution and development teams should decide the optimal frequency for performing SAST and maybe deploy multiple tactics—to balance productivity with adequate security coverage.

 

**Useful Links**:

[Microsoft Security Risk Detection (MSRD)](https://www.microsoft.com/en-us/security-risk-detection/)

[Microsoft DevSkim](https://github.com/Microsoft/DevSkim)

[Roslyn Rules](https://dotnet-security-guard.github.io/rules.htm)

[VSTS security marketplace](https://marketplace.visualstudio.com/search?term=security&target=AzureDevOps&category=All categories&sortBy=Relevance)

[Code Analysis for C/C++](https://msdn.microsoft.com/en-us/library/ms182025.aspx)

[BinSkim](https://github.com/Microsoft/binskim#getting-started-as-a-user)

[FxCop ](https://docs.microsoft.com/en-us/visualstudio/code-quality/install-fxcop-analyzers?view=vs-2017)(legacy tool)

[List of tools for static code analysis](https://en.wikipedia.org/wiki/List_of_tools_for_static_code_analysis) (Wikipedia)

 

### Practice #10 - Perform Dynamic Analysis Security Testing (DAST)

Performing run-time verification of your fully compiled or packaged software checks functionality that is only apparent when all components are integrated and running. This is typically achieved using a tool or suite of prebuilt attacks or tools that specifically monitor application behavior for memory corruption, user privilege issues, and other critical security problems. Similar to SAST, there is no one-size-fits-all solution and while some tools, such as web app scanning tools, can be more readily integrated into the continuous integration / continuous delivery pipeline, other DAST testing such as fuzzing requires a different approach.

 

**Useful Links**:

[VSTS security marketplace](https://marketplace.visualstudio.com/search?term=security&target=AzureDevOps&category=All categories&sortBy=Relevance)

[Automated Penetration Testing with White-Box Fuzzing](https://msdn.microsoft.com/library/cc162782.aspx)

### Practice #11 - Perform Penetration Testing

Penetration testing is a security analysis of a software system performed by skilled security professionals simulating the actions of a hacker. The objective of a penetration test is to uncover potential vulnerabilities resulting from coding errors, system configuration faults, or other operational deployment weaknesses, and as such the test typically finds the broadest variety of vulnerabilities. Penetration tests are often performed in conjunction with automated and manual code reviews to provide a greater level of analysis than would ordinarily be possible.

 

**Useful Links**:

[Attack Surface Analyzer](https://github.com/microsoft/attacksurfaceanalyzer)

[SDL Security Bug Bar Sample](https://msdn.microsoft.com/library/cc307404.aspx)

 

### Practice #12 - Establish a Standard Incident Response Process 

Preparing an Incident Response Plan is crucial for helping to address new threats that can emerge over time. It should be created in coordination with your organization’s dedicated Product Security Incident Response Team (PSIRT). The plan should include who to contact in case of a security emergency, and establish the protocol for security servicing, including plans for code inherited from other groups within the organization and for third-party code. The incident response plan should be tested before it is needed!

 

**Useful Links**:

[Microsoft Incident Response Reference Guide](https://info.microsoft.com/INCIDENT-RESPONSE-REFERENCE-GUIDE.html)

[Using Azure Security Center for an incident response](https://docs.microsoft.com/en-us/azure/security-center/security-center-incident-response)

[Office 365 Security Incident Management](http://download.microsoft.com/download/2/F/1/2F16A9CA-8D4F-4BB5-8F85-3A362131A95B/Office 365 Security Incident Management.pdf)

[Microsoft Incident Response and shared responsibility for cloud computing](https://azure.microsoft.com/en-us/blog/microsoft-incident-response-and-shared-responsibility-for-cloud-computing/)

[Microsoft Security Response Center](https://www.microsoft.com/en-us/msrc)