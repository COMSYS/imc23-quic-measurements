# ECN with QUIC: Challenges in the Wild Code and Dataset
This repository contains an overview of the tools used for our paper [ECN with QUIC: Challenges in the Wild](https://doi.org/10.1145/3618257.3624821).
## Publication
- Constantin Sander, Ike Kunze, Leo Blöcher, Mike Kosek, Klaus Wehrle: ECN with QUIC: Challenges in the Wild. In Proceedings of the 2023 ACM Internet Measurement Conference (IMC'23), 2023.

If you use any portion of our work, please consider citing our publication.
```
@Inproceedings { 2023-sander-quic-ecn,
   title = {ECN with QUIC: Challenges in the Wild},
   year = {2023},
   month = {10},
   publisher = {ACM},
   booktitle = {Proceedings of the Internet Measurement Conference (IMC '23)},
   DOI = {10.1145/3618257.3624821},
   author = {Sander, Constantin and Kunze, Ike and Bl{\"o}cher, Leo and Kosek, Mike and Wehrle, Klaus}
}
```
## Code
In our paper we rely on two tools: Our adapted zgrab2 banner grabber including QUIC/HTTP/3 requests and our quic-ecn-tracebox tracebox implementation for tracing ECN changes along network paths.
These can be found here:
- quic-zgrab2: https://github.com/COMSYS/quic-zgrab2
  - Please mind that this version also uses our adapted quic-go QUIC stack https://github.com/COMSYS/quic-go
- quic-ecn-tracebox: https://github.com/COMSYS/quic-ecn-tracebox

quic-zgrab2 can use the input of the zdns DNS lookup tool and performs TCP and QUIC based HTTP requests.
These results can be fed into ecn-tracebox which depending on the zgrab results decides for further ECN tracing.
Example zgrab2 pipeline:

```
echo 'google.com,{"comsys-source":"toplists-A-www","comsys-date":"2023-09-13","comsys-tool":"DNS-A-www","zones":["tranco","umbrella","majestic"]}' |\
zdns A -name-servers=1.1.1.1 -prefix "www." -metadata-passthrough | \
jq -rc '(.metadata | fromjson) as $m | del(.metadata) | $m + .' | \
./zgrab2/cmd/zgrab2/zgrab2 http --input-format=zdns --max-redirects=3 --use-https --max-size=10 --port=443 --next-protos="h2,http/1.1" --user-agent="Mozilla/5.0 researchscan.comsys.rwth-aachen.de" --always-try-h3 --ecn-mode-h3=ect0 | \
quic-ecn-tracebox -tracers 1 -sample 0.2
```

## Dataset
We publish our toplist-based measurement dataset used in our paper [here](https://doi.org/10.5281/zenodo.8338470).
It contains our quic-ecn-tracebox results (deduplicated by IP per week) and our zgrab2 results.
Please note that the tracebox deduplication also includes results from the prior CZDS run, which we filtered to only include the toplist results due to CZDS's proprietary license.
