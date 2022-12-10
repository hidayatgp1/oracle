# Mina zkApp: Oracle

This is tutorial to build Oracle of Mina Protocol Project.

## Make New Project
```sh
zk project oracle
```
```sh
cd oracle
```
Delete few files below
```sh
rm src/Add.ts
rm src/Add.test.ts
rm src/interact.ts
```
Edit *index.ts* file in *src* folder
```sh
import { OracleExample } from './OracleExample.js';

export { OracleExample };
```
### Create new file in *src* folder with name *CreditScoreOracle.ts* and write
```sh
import {
    Field,
    SmartContract,
    state,
    State,
    method,
    DeployArgs,
    Permissions,
    PublicKey,
    Signature,
    PrivateKey,
  } from 'snarkyjs';
  
  // The public key of our trusted data provider
  const ORACLE_PUBLIC_KEY =
    'B62qoAE4rBRuTgC42vqvEyUqCGhaZsW58SKVW4Ht8aYqP9UTvxFWBgy';
  
  export class OracleExample extends SmartContract {
    // Define contract state
    @state(PublicKey) oraclePublicKey = State<PublicKey>();
  
    // Define contract events
    events = {
        verified: Field,
      };
  
    deploy(args: DeployArgs) {
      super.deploy(args);
      this.setPermissions({
        ...Permissions.default(),
        editState: Permissions.proofOrSignature(),
      });
    }
  
    @method init(zkappKey: PrivateKey) {
      super.init(zkappKey);
      // Initialize contract state
      this.oraclePublicKey.set(PublicKey.fromBase58(ORACLE_PUBLIC_KEY));
  
      // Specify that caller should include signature with tx instead of proof
      this.requireSignature();
    }
  
    @method verify(id: Field, creditScore: Field, signature: Signature) {
      // Get the oracle public key from the contract state
      const oraclePublicKey = this.oraclePublicKey.get();
      this.oraclePublicKey.assertEquals(oraclePublicKey);
  
      // Evaluate whether the signature is valid for the provided data
      const validSignature = signature.verify(oraclePublicKey, [id, creditScore]);
  
      // Check that the signature is valid
      validSignature.assertTrue();
  
      // Check that the provided credit score is greater than 700
      creditScore.assertGte(Field(700));
  
      // Emit an event containing the verified users id
      this.emitEvent('verified', id);
  
    }
  }
```
### Create new file in *src* folder with name *OracleExampleScaffold.ts* and write
```sh
import {
Field,
SmartContract,
state,
State,
method,
DeployArgs,
Permissions,
PublicKey,
Signature,
PrivateKey,
} from 'snarkyjs';

// The public key of our trusted data provider
const ORACLE_PUBLIC_KEY =
'B62qoAE4rBRuTgC42vqvEyUqCGhaZsW58SKVW4Ht8aYqP9UTvxFWBgy';

export class OracleExample extends SmartContract {
// Define contract state

// Define contract events

deploy(args: DeployArgs) {
  super.deploy(args);
  this.setPermissions({
    ...Permissions.default(),
    editState: Permissions.proofOrSignature(),
  });
}

@method init(zkappKey: PrivateKey) {
  super.init(zkappKey);
  // Initialize contract state

  // Specify that caller should include signature with tx instead of proof
  this.requireSignature();
}

@method verify(id: Field, creditScore: Field, signature: Signature) {
  // Get the oracle public key from the contract state

  // Evaluate whether the signature is valid for the provided data

  // Check that the signature is valid

  // Check that the provided credit score is greater than 700

  // Emit an event containing the verified users id
  
}
}
```
### Create new file in *src* folder with name *OracleExample.ts* and write
```sh
import {
  Field,
  SmartContract,
  state,
  State,
  method,
  DeployArgs,
  Permissions,
  PublicKey,
  Signature,
  PrivateKey,
} from 'snarkyjs';

// The public key of our trusted data provider
const ORACLE_PUBLIC_KEY =
  'B62qoAE4rBRuTgC42vqvEyUqCGhaZsW58SKVW4Ht8aYqP9UTvxFWBgy';

export class OracleExample extends SmartContract {
  // Define contract state
  @state(PublicKey) oraclePublicKey = State<PublicKey>();

  // Define contract events
  events = {
    verified: Field,
  };

  deploy(args: DeployArgs) {
    super.deploy(args);
    this.setPermissions({
      ...Permissions.default(),
      editState: Permissions.proofOrSignature(),
    });
  }

  @method init(zkappKey: PrivateKey) {
    super.init(zkappKey);
    // Initialize contract state
    this.oraclePublicKey.set(PublicKey.fromBase58(ORACLE_PUBLIC_KEY));
    // Specify that caller should include signature with tx instead of proof
    this.requireSignature();
  }

  @method verify(id: Field, creditScore: Field, signature: Signature) {
    // Get the oracle public key from the contract state
    const oraclePublicKey = this.oraclePublicKey.get();
    this.oraclePublicKey.assertEquals(oraclePublicKey);
    // Evaluate whether the signature is valid for the provided data
    const validSignature = signature.verify(oraclePublicKey, [id, creditScore]);
    // Check that the signature is valid
    validSignature.assertTrue();
    // Check that the provided credit score is greater than 700
    creditScore.assertGte(Field(700));
    // Emit an event containing the verified users id
    this.emitEvent('verified', id);
  }
}
```
### Create new file in *src* folder with name *OracleExample.test.ts* and write
```sh
import { OracleExample } from './OracleExample';
import {
  isReady,
  shutdown,
  Field,
  Mina,
  PrivateKey,
  PublicKey,
  AccountUpdate,
  Signature,
} from 'snarkyjs';

// The public key of our trusted data provider
const ORACLE_PUBLIC_KEY =
  'B62qoAE4rBRuTgC42vqvEyUqCGhaZsW58SKVW4Ht8aYqP9UTvxFWBgy';

let proofsEnabled = false;
function createLocalBlockchain() {
  const Local = Mina.LocalBlockchain({ proofsEnabled });
  Mina.setActiveInstance(Local);
  return Local.testAccounts[0].privateKey;
}

async function localDeploy(
  zkAppInstance: OracleExample,
  zkAppPrivatekey: PrivateKey,
  deployerAccount: PrivateKey
) {
  const txn = await Mina.transaction(deployerAccount, () => {
    AccountUpdate.fundNewAccount(deployerAccount);
    zkAppInstance.deploy({ zkappKey: zkAppPrivatekey });
    zkAppInstance.init(zkAppPrivatekey);
  });
  await txn.prove();
  txn.sign([zkAppPrivatekey]);
  await txn.send();
}

describe('CreditScoreOracle', () => {
  let deployerAccount: PrivateKey,
    zkAppAddress: PublicKey,
    zkAppPrivateKey: PrivateKey;

  beforeAll(async () => {
    await isReady;
    if (proofsEnabled) OracleExample.compile();
  });

  beforeEach(async () => {
    deployerAccount = createLocalBlockchain();
    zkAppPrivateKey = PrivateKey.random();
    zkAppAddress = zkAppPrivateKey.toPublicKey();
  });

  afterAll(async () => {
    // `shutdown()` internally calls `process.exit()` which will exit the running Jest process early.
    // Specifying a timeout of 0 is a workaround to defer `shutdown()` until Jest is done running all tests.
    // This should be fixed with https://github.com/MinaProtocol/mina/issues/10943
    setTimeout(shutdown, 0);
  });

  it('generates and deploys the `CreditScoreOracle` smart contract', async () => {
    const zkAppInstance = new OracleExample(zkAppAddress);
    await localDeploy(zkAppInstance, zkAppPrivateKey, deployerAccount);
    const oraclePublicKey = zkAppInstance.oraclePublicKey.get();
    expect(oraclePublicKey).toEqual(PublicKey.fromBase58(ORACLE_PUBLIC_KEY));
  });

  describe('actual API requests', () => {
    it('emits an `id` event containing the users id if their credit score is above 700 and the provided signature is valid', async () => {
      const zkAppInstance = new OracleExample(zkAppAddress);
      await localDeploy(zkAppInstance, zkAppPrivateKey, deployerAccount);

      const response = await fetch(
        'https://mina-credit-score-signer-pe3eh.ondigitalocean.app/user/1'
      );
      const data = await response.json();

      const id = Field(data.data.id);
      const creditScore = Field(data.data.creditScore);
      const signature = Signature.fromJSON(data.signature);

      const txn = await Mina.transaction(deployerAccount, () => {
        zkAppInstance.verify(
          id,
          creditScore,
          signature ?? fail('something is wrong with the signature')
        );
      });
      await txn.prove();
      await txn.send();

      const events = await zkAppInstance.fetchEvents();
      const verifiedEventValue = events[0].event.toFields(null)[0];
      expect(verifiedEventValue).toEqual(id);
    });

    it('throws an error if the credit score is below 700 even if the provided signature is valid', async () => {
      const zkAppInstance = new OracleExample(zkAppAddress);
      await localDeploy(zkAppInstance, zkAppPrivateKey, deployerAccount);

      const response = await fetch(
        'https://mina-credit-score-signer-pe3eh.ondigitalocean.app/user/2'
      );
      const data = await response.json();

      const id = Field(data.data.id);
      const creditScore = Field(data.data.creditScore);
      const signature = Signature.fromJSON(data.signature);

      expect(async () => {
        await Mina.transaction(deployerAccount, () => {
          zkAppInstance.verify(
            id,
            creditScore,
            signature ?? fail('something is wrong with the signature')
          );
        });
      }).rejects;
    });
  });

  describe('hardcoded values', () => {
    it('emits an `id` event containing the users id if their credit score is above 700 and the provided signature is valid', async () => {
      const zkAppInstance = new OracleExample(zkAppAddress);
      await localDeploy(zkAppInstance, zkAppPrivateKey, deployerAccount);

      const id = Field(1);
      const creditScore = Field(787);
      const signature = Signature.fromJSON({
        r: '13209474117923890467777795933147746532722569254037337512677934549675287266861',
        s: '12079365427851031707052269572324263778234360478121821973603368912000793139475',
      });

      const txn = await Mina.transaction(deployerAccount, () => {
        zkAppInstance.verify(
          id,
          creditScore,
          signature ?? fail('something is wrong with the signature')
        );
      });
      await txn.prove();
      await txn.send();

      const events = await zkAppInstance.fetchEvents();
      const verifiedEventValue = events[0].event.toFields(null)[0];
      expect(verifiedEventValue).toEqual(id);
    });

    it('throws an error if the credit score is below 700 even if the provided signature is valid', async () => {
      const zkAppInstance = new OracleExample(zkAppAddress);
      await localDeploy(zkAppInstance, zkAppPrivateKey, deployerAccount);

      const id = Field(2);
      const creditScore = Field(536);
      const signature = Signature.fromJSON({
        r: '25163915754510418213153704426580201164374923273432613331381672085201550827220',
        s: '20455871399885835832436646442230538178588318835839502912889034210314761124870',
      });

      expect(async () => {
        await Mina.transaction(deployerAccount, () => {
          zkAppInstance.verify(
            id,
            creditScore,
            signature ?? fail('something is wrong with the signature')
          );
        });
      }).rejects;
    });

    it('throws an error if the credit score is above 700 and the provided signature is invalid', async () => {
      const zkAppInstance = new OracleExample(zkAppAddress);
      await localDeploy(zkAppInstance, zkAppPrivateKey, deployerAccount);

      const id = Field(1);
      const creditScore = Field(787);
      const signature = Signature.fromJSON({
        r: '26545513748775911233424851469484096799413741017006352456100547880447752952428',
        s: '7381406986124079327199694038222605261248869991738054485116460354242251864564',
      });

      expect(async () => {
        await Mina.transaction(deployerAccount, () => {
          zkAppInstance.verify(
            id,
            creditScore,
            signature ?? fail('something is wrong with the signature')
          );
        });
      }).rejects;
    });
  });
});
```
## Run Build
```sh
npm run build
```
## Update Browserlist
```sh
npx browserslist@latest --update-db
```
##Link with Repository Github
Make new repository on Github and run
```sh
git remote add origin *<your-repo-url>*
git push -u origin main
```
## Run The Test
Run The Test with Command
```sh
npm run test
```
## 
## OUTPUT
You might see something like this in output, thet it means you done!

![image](https://user-images.githubusercontent.com/82062683/206862707-65791167-a718-4e42-962c-883994db4df9.png)

## License

[Apache-2.0](LICENSE)
