Insert or update an element in map `m`:

m[key] = elem

Retrieve an element:

elem = m[key]

Delete an element:

delete(m, key)

Test that a key is present with a two-value assignment:

elem, ok = m[key]

If `key` is in `m`, `ok` is `true`. If not, `ok` is `false`.

If `key` is not in the map, then `elem` is the zero value for the map's element type.

**Note:** If `elem` or `ok` have not yet been declared you could use a short declaration form:

elem, ok := m[key]
```go 
package main

import "fmt"

type my_struct struct{
 str string
some_int int
}

func main() {
	m := make(map[string]int)

	m["Answer"] = 42
	fmt.Println("The value:", m["Answer"])

	m["Answer"] = 48
	fmt.Println("The value:", m["Answer"])

	delete(m, "Answer")
	fmt.Println("The value:", m["Answer"])

	v, ok := m["Answer"]
	fmt.Println("The value:", v, "Present?", ok)
	
	my_map:=make(map[string]my_struct)
	my_map["you"]=my_struct{
	"lol", 67}
	fmt.Println(my_map["you"])
	res,cond :=my_map["poop"]
	fmt.Println(res,cond)
}


The value: 42
The value: 48
The value: 0
The value: 0 Present? false
{lol 67}
{ 0} false
```