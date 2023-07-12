# 🚧 Stage 03 - Step 01 - Metal as a Service 🚧
* Runs on AWX
* Creates the MAAS infrastructure, migrates a copy of the MAAS databases to itself, creates the second and third MAAS nodes using that new database, migrates Genesis to 10.24.137.137, then removes MAAS from it and creates the proper first MAAS node at its correct IP and sets up the final MAAS nodes.