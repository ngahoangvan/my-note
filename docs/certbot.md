# Run certbot to create Lets Encrypt certifications for wildcard domain

certbot certonly --manual --preferred-challenges=dns -d *.jhoangv.com
