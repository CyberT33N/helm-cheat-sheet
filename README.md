# Helm Cheat Sheet
Helm Cheat Sheet with the most needed stuff..


<br><br>


## Alle Releases auflisten
```bash
helm list -n namespaceNameHere
```
  - Wenn man den Namespace hier nicht findet helfen auch NICHT die Helm Rollback und Uninstall Befehle um es zu fixen

<br><br>

## Rollback zu der letzten Revision:
```bash
helm --namespace namespaceNameHere rollback platform
```

## Uninstall Platform:
 - Error: UPGRADE FAILED: another operation (install/upgrade/rollback) is in progress

```bash
helm --namespace namespaceNameHere delete platformNameHere
```
