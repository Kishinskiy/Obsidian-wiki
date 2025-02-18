 ```sh
nuget source add \
-name repos.nexus-dev \
-source https://repos.clearwayintegration.com/repository/nuget-dev/index.json \
-UserName ci_user \
-Password "huGxyb-maxvik-0mocpy"
```
```
```sh
nuget list -s repos.nexus-dev
```
```sh
nuget push ITC.API.InfraCommon-null.1.0.1.19.nupkg -source repos.nexus-dev>)
```
