# MVC nLPD – Microsoft 365

Bundle « Minimum Viable de Conformité » (MVC) nLPD pour PME suisses sur
Microsoft 365 Business Premium + Purview Suite.

Site publié : **https://doit4everyone.github.io/mvc-nlpd-m365/**

## Structure

```
docs/                              ← source du site (racine de publication Pages)
├── _config.yml
├── index.md                       ← page d'accueil du bundle (4 guides)
├── purview/
│   ├── index.md                   ← landing Purview
│   ├── procedure-deploiement.md   ← guide consultant (déploiement complet)
│   └── guide-exploitation.md      ← guide client (maintien quotidien)
├── entra-id-p1/
│   └── index.md
├── intune-defender/
│   └── index.md
├── copilot-governance/
│   └── index.md
├── aitm-cookie-hijacking/
│   └── index.md
├── downloads/                     ← .docx et .pdf téléchargeables
│   ├── docx/
│   └── pdf/
└── assets/                        ← captures terrain et ressources
```

Versions anglaises à venir : fichier marqué `EN` déposé à côté de son
équivalent français (ex. `purview/guide-exploitation-en.md`).

## Activation de GitHub Pages

Settings → Pages → Source : **Deploy from a branch** → branche `main` → dossier **`/docs`**.

## Convention de nommage des fichiers téléchargeables

`MVC-nLPD_<NN>-<Sujet>_<Document>_v<version>.<ext>`

Exemples :
- `MVC-nLPD_01-Purview_Procedure_v1.2.docx`
- `MVC-nLPD_01-Purview_Exploitation_v1.2.docx`
- `MVC-nLPD_02-Entra-ID-P1_Procedure_v3.1.docx`
