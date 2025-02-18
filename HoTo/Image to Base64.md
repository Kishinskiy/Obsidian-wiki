
# Encode

## On Linux

Single line result:

```bash
base64 -w 0 DSC_0251.JPG
```

For `HTML`:

```bash
echo "data:image/jpeg;base64,$(base64 -w 0 DSC_0251.JPG)"
```

As file:

```bash
base64 -w 0 DSC_0251.JPG > DSC_0251.JPG.base64
```

In variable:

```bash
IMAGE_BASE64="$(base64 -w 0 DSC_0251.JPG)"
```

In variable for `HTML`:

```bash
IMAGE_BASE64="data:image/jpeg;base64,$(base64 -w 0 DSC_0251.JPG)"
```

## On OSX

On _OSX_, the `base64` binary is different, and the parameters are different. If you want to use it on _OSX_, you should remove `-w 0`.

Single line result:

```bash
base64 DSC_0251.JPG
```

For `HTML`:

```bash
echo "data:image/jpeg;base64,$(base64 DSC_0251.JPG)"
```

As file:

```bash
base64 DSC_0251.JPG > DSC_0251.JPG.base64
```

In variable:

```bash
IMAGE_BASE64="$(base64 DSC_0251.JPG)"
```

In variable for `HTML`:

```bash
IMAGE_BASE64="data:image/jpeg;base64,$(base64 DSC_0251.JPG)"
```

## Generic OSX/Linux

### As Shell Function

```bash
@base64() {
  if [[ "${OSTYPE}" = darwin* ]]; then
    # OSX
    if [ -t 0 ]; then
      base64 "$@"
    else
      cat /dev/stdin | base64 "$@"
    fi
  else
    # Linux
    if [ -t 0 ]; then
      base64 -w 0 "$@"
    else
      cat /dev/stdin | base64 -w 0 "$@"
    fi
  fi
}

# Usage
@base64 DSC_0251.JPG
cat DSC_0251.JPG | @base64
```

### As Shell Script

Create `base64.sh` file with following content:

```bash
#!/usr/bin/env bash
if [[ "${OSTYPE}" = darwin* ]]; then
  # OSX
  if [ -t 0 ]; then
    base64 "$@"
  else
    cat /dev/stdin | base64 "$@"
  fi
else
  # Linux
  if [ -t 0 ]; then
    base64 -w 0 "$@"
  else
    cat /dev/stdin | base64 -w 0 "$@"
  fi
fi
```

Make it executable:

```bash
chmod a+x base64.sh
```

Usage:

```bash
./base64.sh DSC_0251.JPG
cat DSC_0251.JPG | ./base64.sh
```

# Decode

Get you readable data back:

```bash
base64 -d DSC_0251.base64 > DSC_0251.JPG 
```

--

## UPDATE - On MacOS Monterey or the latest

```bash
# Instead of base64 DSC_0251.JPG
base64 -i DSC_0251.JPG
```

If you want to save the output in a file,

```bash
# Instead of base64 -w 0 DSC_0251.JPG > DSC_0251.JPG.base64
base64 -i DSC_0251.JPG -o DSC_0251.JPG.base64
```