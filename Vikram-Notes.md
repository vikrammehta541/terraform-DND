>.\terraform destroy -target="aws_instance.my-ec2-instance"
>for list variable use variable "aws_instance_type_list" and same for map with string
>if used <count> the update output to list output for each instance like value = [for instance in aws_instance.my-ec2-instance : instance.public_ip]
>for_each, can be used in resouce block, module and output
>for_each, can be used with map, and toset()
>tomap(), only used in output
>remeber each.key and each.value

resource "aws_instance" "example" {
  for_each = <map or toset(list)>
  ...
}

>for_each with a Map
resource "aws_instance" "env_ec2" {
  for_each = {
    dev  = "t2.micro"
    prod = "t3.large"
  }

  instance_type = each.value
  tags = {
    Name = each.key
  }
}

>for_each with a Set (use toset())

variable "az_list" {
  default = ["us-east-1a", "us-east-1b"]
}

resource "aws_instance" "az_vm" {
  for_each = toset(var.az_list)

  availability_zone = each.key
  instance_type     = "t2.micro"
  tags = {
    Name = "vm-${each.key}"
  }
}

>for — Loop to extract values

output "instance_dns_list" {
  value = [
    for inst in aws_instance.env_ec2 : inst.public_dns
  ]
}

>toset() — Convert list ➜ set (no duplicates)

output "instance_types_set" {
  value = toset([
    for inst in aws_instance.env_ec2 : inst.instance_type
  ])
}

>tomap() — Convert loop ➜ map (key ➜ value)

output "env_to_dns_map" {
  value = tomap({
    for env, inst in aws_instance.env_ec2 : env => inst.public_dns
  })
}

===============================================






count Meta-Argument — Easy Formula
✅ Easy-to-Remember Formula:

resource "<type>" "<name>" {
  count = <number>
  # other resource config
  tags = {
    Name = "<custom-name-prefix>${count.index}"
  }
}

🔁 Plug in:

    <type> = aws_instance

    <name> = myec2vm

    <number> = 2

    <custom-name-prefix> = "Count-Demo-"

✅ Final Code:

resource "aws_instance" "myec2vm" {
  count         = 2
  instance_type = "t3.micro"
  tags = {
    Name = "Count-Demo-${count.index}"
  }
}

🧠 Remember:

    count = how many copies

    count.index = 0, 1, 2... use for naming

📃 List & Map Variable Access — Easy Formula
✅ List Access Formula:

var.<list_name>[<index>]

Example:

var.instance_type_list[1]  # => "t3.small"

✅ Map Access Formula:

var.<map_name>["<key>"]

Example:

var.instance_type_map["prod"]  # => "t3.large"

🧠 Remember:

    Lists = ordered → use numbers

    Maps = named → use keys

🔁 For Loop with List — Easy Formula
✅ Easy-to-Remember Formula:

output "<name>" {
  value = [for <item> in <resource> : <item>.<attribute>]
}

🔁 Plug in:

    <name> = for_output_list

    <item> = instance

    <resource> = aws_instance.myec2vm

    <attribute> = public_dns

✅ Final Code:

output "for_output_list" {
  value = [for instance in aws_instance.myec2vm : instance.public_dns]
}

🔁 For Loop with Map — Easy Formula
✅ Easy-to-Remember Formula:

output "<name>" {
  value = {for <item> in <resource> : <key_expr> => <value_expr>}
}

🔁 Plug in:

    <name> = for_output_map1

    <item> = instance

    <resource> = aws_instance.myec2vm

    <key_expr> = instance.id

    <value_expr> = instance.public_dns

✅ Final Code:

output "for_output_map1" {
  value = {for instance in aws_instance.myec2vm : instance.id => instance.public_dns}
}

🔁 For Loop with Map – Advanced
✅ Formula:

output "<name>" {
  value = {for <key>, <value> in <resource> : <key> => <value>.<attribute>}
}

Plug in:

    <name> = for_output_map2

    <key> = c

    <value> = instance

    <attribute> = public_dns

✅ Final Code:

output "for_output_map2" {
  value = {for c, instance in aws_instance.myec2vm : c => instance.public_dns}
}

💥 Splat Operator — Easy Formula
✅ Legacy:

<resource>.*.<attribute>

✅ Modern (Preferred):

<resource>[*].<attribute>

Example:

aws_instance.myec2vm[*].public_dns

🧠 Remember:

    [*].attr = grab all values of attr from multiple resources

🌐 For-Each EC2 per AZ with toset — Easy Formula
✅ for_each Formula:

resource "<type>" "<name>" {
  for_each = toset(<list>)
  availability_zone = each.key
}

Plug in:

    <type> = aws_instance

    <name> = myec2vm

    <list> = data.aws_availability_zones.my_azones.names

✅ Final Code:

resource "aws_instance" "myec2vm" {
  for_each = toset(data.aws_availability_zones.my_azones.names)
  availability_zone = each.key
}

📤 Output with toset() — Easy Formula
✅ Formula:

output "<name>" {
  value = toset([for <item> in <resource> : <item>.<attribute>])
}

Plug in:

    <name> = instance_publicip

    <item> = myec2vm

    <resource> = aws_instance.myec2vm

    <attribute> = public_ip

✅ Final Code:

output "instance_publicip" {
  value = toset([for myec2vm in aws_instance.myec2vm : myec2vm.public_ip])
}

🗺️ Output with tomap() — Easy Formula
✅ Formula:

output "<name>" {
  value = tomap({
    for <key>, <value> in <resource> : <key> => <value>.<attribute>
  })
}

Plug in:

    <name> = instance_publicdns2

    <key> = s

    <value> = myec2vm

    <attribute> = public_dns

✅ Final Code:

output "instance_publicdns2" {
  value = tomap({
    for s, myec2vm in aws_instance.myec2vm : s => myec2vm.public_dns
  })
}

🧠 TL;DR Summary Table
Formula Type	Template
count Resource	count = <number> + "${count.index}" for unique names
List Access	var.<list>[<index>]
Map Access	var.<map>["<key>"]
For Loop (List)	[for <x> in <resource> : <x>.<attribute>]
For Loop (Map)	{for <x> in <resource> : <key> => <value>}
Advanced Map Loop	{for <k>, <v> in <resource> : <k> => <v>.<attr>}
Splat (Modern)	<resource>[*].<attribute>
for_each with set	for_each = toset(<list>) + each.key
Output with toset()	toset([for <x> in <res> : <x>.<attr>])
Output with tomap()	tomap({for <k>, <v> in <res> : <k> => <v>.<attr>})

for_each in Terraform — Easy-to-Remember Formula 💡
✅ Basic for_each Formula (with set or map)

resource "<type>" "<name>" {
  for_each = <collection>  # toset(...) or map

  <some_property> = each.key
  # OR
  <some_property> = each.value

  tags = {
    Name = "<prefix>-${each.key}"  # great for naming
  }
}

✅ Example 1: for_each with Set (using toset())
📦 Goal: Create one EC2 instance per Availability Zone
Step-by-Step Plug-In:

    <type> = aws_instance

    <name> = myec2vm

    <collection> = toset(data.aws_availability_zones.available.names)

    <some_property> = availability_zone

    <prefix> = For-Each-Demo

✅ Final Code:

resource "aws_instance" "myec2vm" {
  ami           = data.aws_ami.amzlinux2.id
  instance_type = var.instance_type
  key_name      = var.instance_keypair

  for_each = toset(data.aws_availability_zones.available.names)
  availability_zone = each.key

  tags = {
    Name = "For-Each-Demo-${each.key}"
  }
}

✅ Example 2: for_each with Map
📦 Goal: Launch different EC2 instances per environment

variable "env_map" {
  default = {
    dev  = "t3.micro"
    prod = "t3.large"
  }
}

Step-by-Step Plug-In:

    <type> = aws_instance

    <name> = env_instance

    <collection> = var.env_map

    <some_property> = instance_type = each.value

    <prefix> = Env-VM

✅ Final Code:

resource "aws_instance" "env_instance" {
  ami           = data.aws_ami.amzlinux2.id
  instance_type = each.value
  key_name      = var.instance_keypair

  for_each = var.env_map

  tags = {
    Name = "Env-VM-${each.key}"
  }
}

✅ Output with for_each — Use in Combination
🧾 Formula:

output "<name>" {
  value = toset([
    for <item> in <resource> : <item>.<attribute>
  ])
}

✅ Map Output with tomap():

output "<name>" {
  value = tomap({
    for <key>, <item> in <resource> : <key> => <item>.<attribute>
  })
}

🧠 Quick Reference Table
Part	Formula / Meaning
for_each = toset(<list>)	Use when you have a list
for_each = <map>	Use when you want key-value pairing
each.key	Current key (like AZ name or env name)
each.value	Value from the map (like instance type)
Use in tags	"Name" = "prefix-${each.key}"
toset([...])	Convert list of items to a set for output
tomap({...})	Convert looped values to a map for output
🎯 Mnemonic to Remember

    "For Each Thing, Do a Unique Thing!"
    Use each.key for naming, each.value for config.

✅ TL;DR Template

resource "<type>" "<name>" {
  for_each = toset(<list>)  # or just a map

  <some_property> = each.key  # or each.value

  tags = {
    Name = "<prefix>-${each.key}"
  }
}

