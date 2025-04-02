## 3. MISC Challenge: Almost Empty Container

### Challenge Description
> "Oh no! I really have to get the content of my really special file **FLAG.txt**, but I don't know where it is in the container or how to see its content..."

To investigate, we pull and run the container:

```
docker pull mrzerkeur/almost_empty_container:latest
docker run -it mrzerkeur/almost_empty_container:latest /bin/sh
```

### Investigation Process

- The container appears **empty** upon execution.
- Using **Docker history**, we confirm the container runs Linux:

```
docker history mrzerkeur/almost_empty_container:latest --no-trunc
```

- The output reveals an **Entrypoint**: `/main`, indicating something is being executed.

### Extracting the Container's Content

1. **Create a temporary container:**

   ```
   docker run --name temp_container mrzerkeur/almost_empty_container:latest
   ```

2. **Copy the container’s filesystem:**

   ```
   docker cp temp_container:/ ./container_files
   ```

3. **List the extracted files:**

   ```
   ls /tmp/container_files/
   ```
   **Output:**
   ```
   OFAASSkxne  dev  etc  main  proc  sys
   ```

4. **Retrieve the flag:**

   ```
   cat /tmp/container_files/OFAASSkxne/FLAG.txt
   ```

   **Output:**
   ```
   RETRO{N0_B1NARY}
   ```

 **FLAG: RETRO{N0_B1NARY}**

