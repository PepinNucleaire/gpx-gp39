# GP39 Requirements

## Manual

See [here](gp39_operators_manual.pdf)

## GPX for GP39

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<gpx>
<wpt lat="XX.XXXXXXXXXX" lon="X.XXXXXXXXXX">
	<name>XX</name>
	<extensions>
		<GP39Symbol>X</GP39Symbol>
		<FECColor>X</FECColor>
		<GP39Comment></GP39Comment>
		<GP39Flag>1</GP39Flag>
	</extensions>
</wpt>
</gpx>
```

### Name

- Max 8 characters

### GP39 Symbol

```
0 | Circle
1 | Square
2 | Diamond
3 | 1 fish
4 | 2 fishes
5 | 3 fishes
6 | Wreck
7 | Anchor
8 | Skullhead
9 | Flag
```

### GP39 COlor

```
0 | Red
1 | Yellow
2 | Green
3 | Blue
4 | Purple
5 | Black
6 | Brown
```

### GP39 Comment

- Always empty

### GP39 Flag

- Always 1
