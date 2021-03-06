# Using Providers and Modules

## Prebuilt Providers

To improve the general user experience around provider imports and to allow building further abstractions on top of the Terraform provider bindings, a few popular providers are offered as prebuilt packages. At the moment the following providers are built and published to NPM / PyPi on a regular basis automatically.

- [AWS Provider](https://github.com/terraform-cdk-providers/cdktf-provider-aws)
- [Google Provider](https://github.com/terraform-cdk-providers/cdktf-provider-google)
- [Azure Provider](https://github.com/terraform-cdk-providers/cdktf-provider-azurerm)
- [Kubernetes Provider](https://github.com/terraform-cdk-providers/cdktf-provider-kubernetes)
- [Docker Provider](https://github.com/terraform-cdk-providers/cdktf-provider-docker)
- [Github Provider](https://github.com/terraform-cdk-providers/cdktf-provider-github)
- [Null Provider](https://github.com/terraform-cdk-providers/cdktf-provider-null)

Please check the [Terraform CDK Providers](https://github.com/terraform-cdk-providers) organization as well for an up to date list. As these are normal NPM / PyPI packages, they can be used as any other dependency.

e.g. in Typescript / Node:

```
npm install -a @cdktf/provider-aws
```

## Importing Providers and Modules

CDK for Terraform allows you to import Terraform [providers](https://www.terraform.io/docs/providers/index.html) and [modules](https://www.terraform.io/docs/modules/index.html) to your project
using this workflow.

Let's take the TypeScript [example](../getting-started/typescript.md) shown in the getting started guide.
The project has the `main.ts` file that defines the AWS resources that need to be deployed.

```typescript
import { Construct } from 'constructs';
import { App, TerraformStack } from 'cdktf';
import { AwsProvider, Instance } from './.gen/providers/aws';

class MyStack extends TerraformStack {
  constructor(scope: Construct, id: string) {
    super(scope, id);

    new AwsProvider(this, 'aws', {
      region: 'us-east-1'
    });

    new Instance(this, 'Hello', {
      ami: "ami-2757f631",
      instanceType: "t2.micro"
    });
  }
}

const app = new App();
new MyStack(app, 'hello-terraform');
app.synth();
```

The project also has the [cdktf.json](./cdktf-json.md) file that defines what providers and modules are being used by the project.

```bash
vim cdktf.json
```

```json
{
  "language": "typescript",
  "app": "npm run --silent compile && node main.js",
  "terraformProviders": [
    "aws@~> 2.0"
  ]
}
```

In order to use another provider or module, edit the `cdktf.json` file and add the name of the provider.
For example, to add [DNS Simple](https://www.terraform.io/docs/providers/dnsimple/index.html) provider to the project, edit the `cdktf.json` file and add the provider name to the `terraformProviders` array.

```json
{
  "language": "typescript",
  "app": "npm run --silent compile && node main.js",
  "terraformProviders": [
    "aws@~> 2.0",
    "dnsimple"
  ]
}
```

Then run `cdktf get` command in the working directory.

```bash
cdktf get
⠋ downloading and generating providers...
```

```bash
Generated typescript constructs in the output directory: .gen
```

This command creates the appropriate TypeScript classes automatically that can be imported in the application.

Import the `DnsimpleProvider` and `Record` resources from `./.gen/providers/dnsimple` and define them.

```typescript
import { Construct } from 'constructs';
import { App, TerraformStack } from 'cdktf';
import { AwsProvider, Instance } from './.gen/providers/aws';
import { DnsimpleProvider, Record } from './.gen/providers/dnsimple';

class MyStack extends TerraformStack {
  constructor(scope: Construct, id: string) {
    super(scope, id);

    new AwsProvider(this, 'aws', {
      region: 'us-east-1'
    });

    const instance = new Instance(this, 'Hello', {
      ami: "ami-2757f631",
      instanceType: "t2.micro"
    });

    new DnsimpleProvider(this, 'dnsimple', {
      token: Token.asString(process.env.DNSIMPLE_TOKEN),
      account: Token.asString(process.env.DNSIMPLE_ACCOUNT)
    });

    new Record(this, 'web-www', {
      domain: 'example.com',
      name: 'web',
      value: instance.publicIp,
      type: 'A'
    });
  }
}

const app = new App();
new MyStack(app, 'hello-terraform');
app.synth();
```

Synthesize the code.

```bash
cdktf synth --json
```

```json
{
  "//": {
    "metadata": {
      "version": "0.0.11-pre.8757404fa25b6e405f1a51eac11b96943ccb372e",
      "stackName": "vpc-example"
    }
  },
  "terraform": {
    "required_providers": {
      "aws": "~> 2.0",
      "dnsimple": "undefined"
    }
  },
  "provider": {
    "aws": [
      {
        "region": "us-east-1"
      }
    ],
    "dnsimple": [
      {
        "account": "hello@example.com",
        "token": "xxxxxxxxxx"
      }
    ]
  },
  "resource": {
    "aws_instance": {
      "vpcexample_Hello_279554CB": {
        "ami": "ami-2757f631",
        "instance_type": "t2.micro",
        "//": {
          "metadata": {
            "path": "vpc-example/Hello",
            "uniqueId": "vpcexample_Hello_279554CB",
            "stackTrace": [
              .....
            ]
          }
        }
      }
    },
    "dnsimple_record": {
      "vpcexample_webwww_477C7150": {
        "domain": "example.com",
        "name": "web",
        "type": "A",
        "value": "${aws_instance.vpcexample_Hello_279554CB.public_ip}",
        "//": {
          "metadata": {
            "path": "vpc-example/web-www",
            "uniqueId": "vpcexample_webwww_477C7150",
            "stackTrace": [
              .....
            ]
          }
        }
      }
    }
  }
}

```
