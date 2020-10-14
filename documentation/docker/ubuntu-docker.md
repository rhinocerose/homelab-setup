Sourced from [here](https://docs.organizr.app/books/tutorials/page/installing-ubuntu%28or-kubuntu%29-and-installing-docker%28part-1%29) which has interesting `docker-compose`
[here](https://docs.organizr.app/books/tutorials/page/creating-docker-compose-file%28part-3%29)

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable edge"
sudo apt-get update && apt-cache policy docker-ce
sudo apt-get install -y docker-ce
sudo systemctl status docker
sudo usermod -aG docker ${USER}
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /
sudo chmod +x /usr/local/bin/docker-compose && sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
