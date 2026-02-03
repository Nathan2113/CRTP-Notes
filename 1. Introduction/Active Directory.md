# Components

- schema - defines objects and their attributes
- query and index mechanism - provides searching and publication of objects and props
- global catalog - contains info about every object in the directory
- replication service - distributes info across domain controllers


# Structure

- Forests, domains, and OUs are the building blocks of AD structure
	- forest - security boundary, may contain multiple domains and each domain may contain multiple OUs
- always think of AD as a forest, not a domain controller
	- a lot of configurations only work when put on the forest root
	- when a single domain in the forest is compromised, the entire forest is done as well