# Week 1

**clone repo:**

```
git clone git@github.com:input-output-hk/plutus-pioneer-program.git
```

**cabal build:**

```
cd code /week01
cabal build
```

### Run playgorund

#### Run in docker (recommended)

```
git clone git@github.com:maccam912/ppp.git
cd ppp
docker-compose -f docker-compose-week01.yml up
```

go to: https://localhost:8009/

#### Run locally

**Nix SHELL**

https://docs.plutus-community.com/docs/setup/Ubuntu.html

```
cat cabal.project
```

```
source-repository-package
  type: git
  location: https://github.com/input-output-hk/plutus.git
  tag: 3746610e53654a1167aeb4c6294c6096d16b0502
```

**clone plutus in another folder:**

```
cd ..
git clone https://github.com/input-output-hk/plutus.git
cd plutus
```

_check commit:_ `3746610e53654a1167aeb4c6294c6096d16b0502`

```
nix-shell
```

Open **two** consoles with **nix-shell**

1rst. nix-shell (server)

```
cd plutus-playground-client
plutus-playground-server
```

2nd. nix-shell (client)

```
cd plutus-playground-client
npm run start
```

go to: https://localhost:8009/
