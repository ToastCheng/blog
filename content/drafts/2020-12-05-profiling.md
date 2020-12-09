
### pprof
- test
  add flags: `-cpuprofile`, `-memprofile`
- explicitly added
  `runtime.StartCPUProdile`
  `runtime.StartHeapProdile`
- server
  `import _ "net/http/pprof"`
  see in `/debug/pprof`
  or `go tool pprof --seconds 5 http://localhost:9090/debug/pprof`

- frequently used command
  - `top10`
  - `top10 -cum`
  - `list <regex>`
  - `list addTagsToName`
  - `disasm <regex>`

### cpu profile
stack sampling
profiling is to sample the function in the call stack
so if the program execute so fast we not gonna make many sample and we might not see the function in the profile, 

you might see background noise only.
profiling period: 10ms once
`go tool pprof -http=8080 cpu.pprof`


### memory profile

### trace profile


### go-torch
flame graph
