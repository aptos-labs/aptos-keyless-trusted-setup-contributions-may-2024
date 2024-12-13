# WARNING

This repo is very large. Before cloning, please follow the instructions below in "Downloading Files" to avoid downloading unnecessary data. 

# Downloading Files

To download the large `.r1cs`, `.ptau`, and `.zkey` files in this repo, [install git-lfs](https://git-lfs.com/). 

Then run the following commands to clone the repo without the larger files:

```
GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/aptos-labs/aptos-keyless-trusted-setup-contributions-may-2024

cd aptos-keyless-trusted-setup-contributions-may-2024
```

To get a specific file `<filename>`, run

```
git lfs pull --include "<filename>"
```

# Setup

If needed, [install npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm), and also [install nvm](https://github.com/nvm-sh/nvm?tab=readme-ov-file#installing-and-updating). Then run the commands

```
nvm use 18
```

and

```
npm install snarkjs@0.6.11
```

to install the version of `snarkjs` used by our code which was used to run the setup ceremony. 

To verify the hash values of various files presented in this document, ensure [Homebrew is installed](https://docs.brew.sh/Installation) and run 

```
brew install b2sum
```

The `b2sum` hash of a file may be computed by running `b2sum <filename>`.

# Verifying the Phase 1 Powers of Tau File

The powers of tau file used was downloaded from one of the links provided here in the [Snarkjs README](https://github.com/iden3/snarkjs/blob/master/README.md#7-prepare-phase-2), as the file titled `powersOfTau28_hez_final_21.ptau`. This `.ptau` file contains 54 contributions and a random beacon. Its `b2sum` hash is 

```
9aef0573cef4ded9c4a75f148709056bf989f80dad96876aadeb6f1c6d062391f07a394a9e756d16f7eb233198d5b69407cca44594c763ab4a5b67ae73254678
``` 

To verify this file, run 

```
npx snarkjs@0.6.11 powersoftau verify powersOfTau28_hez_final_21.ptau -v
```

# Reproducing the .r1cs file

To reproduce `main.r1cs`, clone the [Aptos Core Repo](https://github.com/aptos-labs/aptos-core), checkout commit `39f9c44b4342ed5e6941fae36cf6c87c52b1e17f`, and run the following commands:

```
cd keyless/circuit-data/templates
circom -l . main.circom --r1cs
```

The `b2sum` hash of the resulting `main.r1cs` file should be 

```
9e9bb596c7a453521e9b8df1dbd82c0bad5788f2679f8c53a6bc6112b9ec6061fea36547efa649ba9af78f6792ecd639a654ea6a1d11553b1ce93998924b3ac2
```

which is identical to the `b2sum` hash of `main_39f9c44b4342ed5e6941fae36cf6c87c52b1e17f.r1cs` in this repo.

This provides a link between our circuit code and the trusted setup, ensuring you can verify the setup was done over the correct codebase. Without this link, you could not know what circuit the setup was done over, making it potentially insecure. 

# Reproducing the initial .zkey file

To reproduce the initial `.zkey` file, run the command

```
npx snarkjs@0.6.11 groth16 setup main.r1cs ./powersOfTau28_hez_final_21.ptau initial.zkey -v
```

The `b2sum` hash of the resulting `.zkey` file should match that of `contributions/main_00000.zkey`. This hash value is

```
8381954aa98bc7c7e11fd1f74f544320937f291fd1ce83cfc2a227c805b69de5362781c2484b125c600591cb54a34da56c0911f9a0fea066636f5c79ddbcc3c4
```



# Verifying Individual Contributions


The folder `contributions` contains all `.zkey` files output by the Aptos OIDB trusted setup. Each `.zkey` file corresponds to the contribution of one participant, in the order of their having participated, so that i.e. `main_00004.zkey` corresponds to the output of the contribution made by the 4-th participant to have completed their contribution.

Each contribution may be verified by running the command 

```
npx snarkjs@0.6.11 zkey verify main.r1cs powersOfTau28_hez_final_21.ptau contributions/<contribution_filename>.zkey -v
```

Depending on your machine, this command can take upwards of 20 minutes. Upon completion, it will produce an output of the following form: 

```
[INFO]  snarkJS: Circuit Hash: 
		0a9aeb1f 247e8652 46a00128 ab2fbe10
		86ad8f2a 86a17f72 bd35f095 dfde1a86
		646f6faa fdaa46d6 9dde1ad5 88f21d4d
		0d4e9e74 519b561d 9e71e768 9c739b68
[INFO]  snarkJS: -------------------------
[INFO]  snarkJS: contribution #3 sherry-x-55457892:
		0125c8cb de936779 25ff1b62 49eff731
		15a68679 899678bb 83a19c0e cf8daf28
		20dcf4a9 78192394 2895cbc0 1ba661b3
		1628f395 f553b409 16428355 cfd53f6b
[INFO]  snarkJS: -------------------------
[INFO]  snarkJS: contribution #2 michaelaptos-158355435:
		63c5d60f 668bd588 d6c86d0e a459f24d
		d75c207a 76dbc07a 36c466c5 97ef3ba7
		760e6dd3 b3162767 313935eb afe8fd59
		1b4868b0 238ee70c beb60e3b a46a38d6
[INFO]  snarkJS: -------------------------
[INFO]  snarkJS: contribution #1 alinush-1724810:
		65c3e44a e8ae9dcd 2406d1b4 7413e09c
		72a6587b 5dbc6e00 ac607c84 cefbc4c1
		a9dd4d2c 39d78381 ea2a8cf2 8d3f4e91
		c32c2e8b eaf17465 b9bc8045 a529ecf1
[INFO]  snarkJS: -------------------------
[INFO]  snarkJS: ZKey Ok!
```

This is a list of all contribution hashes up to and including the current contribution that you have just verified. If you participated in the setup, you can verify that your contribution is being used correctly by running this command on the final `.zkey` file, `contributions/main_final.zkey`, and comparing the output contribution hash corresponding to your contribution to the one which was printed out when you finished contributing during the ceremony. 

This may be done with the following command:

```
npx snarkjs@0.6.11 zkey verify main.r1cs powersOfTau28_hez_final_21.ptau contributions/main_final.zkey -v
```

Note that the contribution hash is not simply a hash of the `.zkey` file, but a hash of the participant-specific parameters stored by every `.zkey` file after a participant's contribution has been made. Verification on the `i`-th `.zkey` file includes checks on these parameters for participants `1` through `i`. The output of this process is used in verifying the `.zkey` file overall when running the above `zkey verify` command. 

# Applying the randomness beacon

To eliminate potential bias, we apply the [Drand randomness beacon](https://drand.love/) to the final `.zkey` file to obtain our final `.zkey` file for use in production. To regenerate our randomness, first [follow the instructions here](https://drand.love/developer/drand-client/#installation) and then run

```
./drand-client --round 3977904 \
--chain-hash 8990e7a9aaed2ffed73dbd7092123d6f289930540d7651336225dc172e51b2ce \
--url http://api.drand.sh \
--relay /dnsaddr/api.drand.sh
```

which should output round number `3977904` and randomness

```
897783163d9f6d305b57b5a47e856a31cd6ee33c9dc8d56d0a59579def57558a
```

You can recreate the final `.zkey` file by running

```
npx snarkjs@0.6.11 zkey beacon contributions/main_00141.zkey main_final.zkey da44fdb1c88a25fd68d8581e077dd9e4d6d4c8af22c30b127a23dd8343995565 10 -v
```

This command may take 10-15 minutes. It should output

```
[INFO]  snarkJS: Contribution Hash: 
		fe2a67c9 267a89ac 401e1e0c f6967df5
		84f10fcd 07f49b71 cd204771 f7eff35f
		f60bc605 195b1912 0b698f0b 15b4247e
		84188144 2c7bd57a 9143de22 430379cd
```

As with other contributions, this `.zkey` file may be verified by running

```
npx snarkjs@0.6.11 zkey verify main.r1cs powersOfTau28_hez_final_21.ptau main_final.zkey -v
```

The `b2sum` hash of the resulting `main_final.zkey` should be 

```
07ee7c4de536efe9866a351001698ffdd2bd377b148bec67e4adadd1bb1d3e09ce7b2f07e68ba207f339e4ee8e44335f9c2c574c07d51aa0283dcc8cb65ad992
```

which is identical to the `b2sum` hash of `contributions/main_final.zkey`.

# Reproducing the verification key

The final verification key can be exported using 

```
npx snarkjs@0.6.11 zkey export verificationkey contributions/main_final.zkey verification_key_regen.json
```

Its `b2sum` hash is

```
57e3e3f7f9a43566b64eef59b8ce868b3c55c12c6ee0b52660006067bbe98c51d20644334d8ae587f54736f6a8b81a539b23017b0abeb3d03046d893faaa691b
```

This should be identical to the `verification_key.json` file already in the repo.

# Participant List

The list of participants who both completed their contribution and consented to have their identity listed are presented below. We would like to sincerely thank both them and those who have chosen to remain anonymous for contributing to the security of Aptos Keyless. 

# aptos-keyless-trusted-setup-contributions-may-2024

```
Daniel Porteous (dport)
Maykon Michel Palma
Max Kaplan
Rasa Welcher
Greg Nazario
Rustie Lin
Rati Gelashvili
zhoujunma
Sherry Xiao
Maayan
Michael Straka
None
Zekun Li
Alyssa Ponzo
Artsiom Holikau
Philip Vu
Eric Carlson
darren
schultzie
Gabriele Della Casa Venturelli
None
John Youngseok Yang
Alin Tomescu
Avery Ching
Oliver He
Kobi Gurkan
jill
Angie Huang
Rex Fernando
Aleks Zi
```
