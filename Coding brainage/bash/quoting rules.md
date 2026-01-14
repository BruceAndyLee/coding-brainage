
| Quotation mark                            | equivalent                  | comment                                                               |
| ----------------------------------------- | --------------------------- | --------------------------------------------------------------------- |
| "use"                                     | use                         |                                                                       |
| `use=3`<br>`"${use}"`                     | "$use"                      | allows for interpolation                                              |
| `destination=3`<br>`ssh "${destination}"` |                             |                                                                       |
| `cmd "some line"`                         |                             | allows for space symbols in the argument                              |
| `ssh "${target} -p ${port}"`              | `ssh "192.168.100.3 -p 20"` | WRONG: the first argument of the ssh command contains both parameters |
| `ssh $target -p $port`                    | `ssh 192.168.100.3 -p 20`   | CORRECT: each argument of the command is passed separately            |
