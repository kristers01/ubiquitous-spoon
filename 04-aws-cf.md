## Cloudformation (CF) template izveide
- [ ] Atver jebkuru vidi, kur rakstīsi kodu (VS Code. Eclipse, IntelliJ IDEA, Notepad++, kas kuram ērtāk) un izveido jaunu failu - `first-stack.yaml`. Jaunajā failā izveido pirmās divas sadaļas - obligāto *AWSTemplateFormatVersion* un neobligāto *Description*:
    ```yaml
    ---
    AWSTemplateFormatVersion: "2010-09-09"

    Description:
    CF template to create the first security group
    ```
- [ ] Pievieno failam *Parameters* sadaļu - šeit var likt parametrus, kurus padod CF stack izveides brīdī:
    ```yaml
    Parameters:
      OwnerName:
        Type: String
        Description: Enter your name (no spaces)
    ```
    Ievēro atkāpes - Parameters ir vienādā līmenī ar Description, bet katrs nākamais līmenis ir par divām atstarpēm (whitespaces) tālāk kā iepriekšējais. YAML, tāpat kā Python, atkāpes ir ļoti nozīmīgas!
- [ ] Pievieno failam obligāto *Resources* sadaļu. Resources apraksta, kādus resursus/servisus AWS mēģinās izveidot un to konfigurāciju. Pagaidām izveidosim tikai Security Group, kam atvērti visi izejošie (outbound) porti un ieejošos portos atļauts 22 un 8080 ports no jebkurienes.
    ```yaml
    Resources:
      FirstSG:
        Type: AWS::EC2::SecurityGroup
        Properties:
          GroupDescription: The first security group
          GroupName: !Sub "${OwnerName}-sg"
          SecurityGroupEgress:
            - IpProtocol: tcp
            FromPort: 0
            ToPort: 65535
            CidrIp: 0.0.0.0/0
          SecurityGroupIngress:
            - IpProtocol: tcp
            FromPort: 22
            ToPort: 22
            CidrIp: 0.0.0.0/0
            Description: Allows SSH from anywhere
            - IpProtocol: tcp
            FromPort: 8080
            ToPort: 8080
            CidrIp: 0.0.0.0/0
            Description: Allows to access Jenkins from anywhere
          VpcId: vpc-04ab2a705425e2060
          Tags:
            - Key: "CreatedBy"
            Value:
                Ref: OwnerName
    ```
    - FirstSG - brīvi izvēlēts resursa identifikators, kā uz šo resursu atsauksies pārējie resursi
    - Type - resursa tips no AWS dokumentācijas
    - Properties nosaka dažādus iestatījumus konkrētajam resursam un sīkāka dokumentācija par katru apskatāma https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html

## Cloudformation template validācija izmantojot linterus
Tagad, kad fails ir gatavs, ir vērts pārbaudīt, vai tajā nav kļūdu. Ir vairāki linteri/validācijas rīki, lai pārbaudītu yaml faila sintaksi un koda noformējumu.
- [ ] Instalē kādu no YAML linteriem un pārbaudi sintaksi `fisrt-stack.yaml` failā, piemēram, ja izmanto:
  - *WSL*, vari izmantot komandrindas rīku *yamllint*:
    1. Atver WSL (Ubuntu/Linux) un atkarībā no distro, instalē yamllint izmantojot komandu:
        ```Shell
        sudo apt-get install yamllint // ubuntu
        sudo dnf install yamllint // linux
        ```
    1. Tad nomaini direktoriju uz vietu, kur atrodas CF template un validē template ar komandu: `yamllint first-stack.yaml` Komanda atgriež rezultātu tikai, ja failā ir kļūdas
  - *pip*, vari validēt failu mazā python kodā izmantojot *pyyaml* pakotni:
    1. Atver termināli un instalē pakotni *pyyaml*, izmantojot komandu: `pip install --user yamllint`
    1. Tad nomaini direktoriju uz vietu, kur atrodas CF template un validē template ar komandu: 
        ```Shell
        python -c 'import yaml, sys; print(yaml.safe_load(sys.stdin))' < first-stack.yaml
        ```
  - *VS Code*, instalē kādu no daudzajiem YAML validēšanas spraudņiem, piemēram, Serverless IDE (https://marketplace.visualstudio.com/items?itemName=ThreadHeap.serverless-ide-vscode) vai cloudformation-yaml-validator (https://marketplace.visualstudio.com/items?itemName=champgm.cloudformation-yaml-validator)
  - *Eclipse*, instalē yaml-editor https://marketplace.eclipse.org/content/yaml-editor
  - *IntelliJ IDEA*, instalē yaml spraudni https://plugins.jetbrains.com/plugin/13126-yaml

## Security Group izveidošana izmantojot Cloudformation
- [ ] Atver AWS konsoli https://509136055285.signin.aws.amazon.com/console un ieej savā kontā:
- [ ] Pārliecinies vai augšējā labā stūrī reģions izvēlēts *Ireland* (Europe (Ireland) eu-west-1). Ja nē, nomaini to uz *Ireland*
- [ ] Atver Cloudformation konsoli meklējot resursu *CloudFormation* un izvēloties *Create Stack* vai atverot https://eu-west-1.console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/create/template, kas atver formu pirmā CF stack izveidei.
  1. *Specify template* solis - pie *Prepare template* izvēlies *Template is ready*, pie *Specify template* - *Upload a template file* -> Choose file un augšuplādē tikko uzrakstīto `first-stack.yaml` failu -> Next
  1. *Specify stack details*
      - Stack name - tavsVards-first-stack
      - OwnerName - tavsVards bez atstarpēm, mīkstinājuma un garumzīmēm
    Spied *Next*
  1. *Configure stack options* - nemaini neko un atstāj standarta iestatījumus - Next
  1. *Review* - pārskati stack detaļas un spied *Create stack*
- [ ] Lai apskatītu progresu, atjauno lapu izmantojot pārlūka vai AWS atjaunošanas pogu labējā augšējā stūrī. Kad statuss ir *CREATE_COMPLETE*, apsveicu, esi izveidojis savu pirmo CloudFormation stack. Spied uz Resources, lai apskatītu kādus resursus stack ir izveidojis. Uzspied uz FirstSG Physical ID, kas jaunā lapā atvērs izveidoto SG. Tā kā ar vienu pašu SG būs par maz, izlabo CF stack, lai tas izveidotu vēl vienu resursu - EC2 instanci

## Cloudformation (CF) template labošana un validācija
- [ ] Atver mīļākajā IDE `first-stack.yaml` failu un veic sekojošus labojumus:
  1. *Parameters* sadaļā pievieno vēl vienu parametru *InstanceTypeParameter* (*InstanceTypeParameter* jabūt vienā līmenī ar *OwnerName*):
    ```yaml
      InstanceTypeParameter:
        Type: String
        Default: t2.micro
        AllowedValues:
          - t2.micro
          - m1.small
        Description: Enter t2.micro or m1.small. Default is t2.micro as it is included in the Free Tier.
    ```
  2. *Resources* sadaļā pievieno bloku *FirstEC2Instance* (*FirstEC2Instance* jābūt vienādā līmenī ar *FirstSG*):
    ```yaml
      FirstEC2Instance:
        Type: "AWS::EC2::Instance"
        Properties:
          IamInstanceProfile: SSMRole
          ImageId: "ami-0ed961fa828560210"
          KeyName: student
          InstanceType:
            Ref: InstanceTypeParameter
          NetworkInterfaces:
            - AssociatePublicIpAddress: True
              DeleteOnTermination: True
              DeviceIndex: "0"
              GroupSet:
                - Ref: FirstSG
              SubnetId: "subnet-0e0716aa107924a84"
          Tags:
            - Key: "CreatedBy"
              Value:
                Ref: OwnerName
            - Key: "Name"
              Value: !Sub "${OwnerName}-cf-instance"
          UserData:
            Fn::Base64:
              !Sub |
                #!/bin/bash -xe
                sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
                sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
                sudo yum upgrade -y
                sudo amazon-linux-extras install epel java-openjdk11 -y
                sudo yum install -y jenkins
                sudo systemctl daemon-reload
                sudo systemctl start jenkins
                sudo systemctl enable jenkins
    ```
    - FirstEC2Instance - brīvi izvēlēts resursa identifikators, kā uz šo resursu atsauksies pārējie resursi
    - Type - resursa tips no AWS dokumentācijas
    - Properties nosaka dažādus iestatījumus konkrētajam resursam un sīkāka dokumentācija par katru apskatāma https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html
  3. Zem *Resources* sadaļas pievieno *Outputs* sadaļu, tai jābūt vienā līmenī ar Resources. Šī sadaļa nav obligāta, bet iekļauj noderīgu informāciju, kā piemēram, kad stack būs izveidots, izvadīsim Publisko IP adresi kopā ar portu, lai piekļūtu Jenkins
    ```yaml
    Outputs:
      JenkinsUrl:
        Description: Jenkins URL
        Value: !Sub
          - http://${PublicIP}:8080
          - { PublicIP: !GetAtt FirstEC2Instance.PublicIp}
    ```
- [ ] Saglabā un validē failu izmantojot kādu no iepriekš instalētajiem un izmantotajiem linteriem. Vai linteris atrod vairāk kļūdu kā iepriekš?

## Jaunu resursu pievienošana CloudFormation (CF) stack 
- [ ] Atver Cloudformation konsoli meklējot resursu *CloudFormation* vai atverot https://eu-west-1.console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks?filteringStatus=active&filteringText=&viewNested=true&hideStacks=false un atrodi savu stack pēc sava vārda, izvēlies savu stack un spied *Update* augšējā labējā stūrī.
  1. *Specify template* - izvēlies *Replace current template* un tad *Upload a template file* -> *Choose file*. Augšuplādē jaunāko `first-stack.yaml` failu, spied *Next*
  1. *Specify stack details* - atstāj parametrus bez izmaiņām, ievēro, ka tagad ir arī izvēles parametrs *InstanceTypeParameter*, atstāj noklusējuma vērtību *t2.micro*. Spied *Next*
  1. *Configure stack options* - nemaini neko un atstāj standarta iestatījumus - Spied *Next*
  1. *Review* - pārskati stack detaļas un spied *Update stack*
- [ ] Lai apskatītu progresu, atjauno lapu izmantojot pārlūka vai AWS atjaunošanas pogu labējā augšējā stūrī. Kad statuss ir nomainījies uz *UPDATE_COMPLETE*, esi atjaunojis savu CloudFormation stack. Spied uz Resources, lai apskatītu kādus resursus stack ir izveidojis. Tagad, papildus *FirstSG* Security group ir izveidota arī instance *FirstEC2Instance*. Aizej uz *Outputs* tab un spied uz JenkinsUrl adreses.
- [ ] Uzstādi Jenkins sekojot instrukcijām tāpat kā 3. laboratorijas darbā (savu instanci atradīsi pie EC2 instances https://eu-west-1.console.aws.amazon.com/ec2/v2/home?region=eu-west-1#Instances: meklējot pēc sava vārda)

## Papildus uzdevums tiem, kas pabeidz ātrāk
- [ ] Veic izmaiņas `first-stack.yaml`, lai:
  1. uz esošās instances uzinstalētu SonarQube
  vai
  2. izveidotu jaunu instanci, kur uzinstalē SonarQube
  Instrukcijas, kā uzstādīt SonarQube, atradīsi šeit - https://docs.sonarqube.org/latest/setup/install-server/

## Resursu tīrīšana
- [ ] Atver Cloudformation konsoli meklējot resursu *CloudFormation* vai atverot https://eu-west-1.console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks?filteringStatus=active&filteringText=&viewNested=true&hideStacks=false un atrodi savu stack pēc sava vārda, izvēlies to un spied *Delete* -> *Delete stack*
Lai apskatītu progresu, atjauno lapu izmantojot pārlūka vai AWS atjaunošanas pogu labējā augšējā stūrī. https://eu-west-1.console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks?filteringStatus=deleted&filteringText=&viewNested=true&hideStacks=false&stackId= meklējot pēc sava vārda vari pārlieicnāties, ka statuss norāda *DELETE_COMPLETE*