# Docker Cheat Sheet

## 📚 Essential Commands

### Images
```bash
docker images         # List images
docker pull image     # Pull image
docker build -t name .  # Build image
docker rmi image      # Remove image
```

### Containers
```bash
docker ps             # Running containers
docker ps -a          # All containers
docker run image      # Run container
docker stop container # Stop container
docker rm container   # Remove container
docker exec -it container bash  # Execute
docker logs container # View logs
```

### Docker Compose
```bash
docker-compose up     # Start services
docker-compose down   # Stop services
docker-compose logs   # View logs
docker-compose ps     # List services
```

---

**Quick Reference**: Essential Docker commands.

