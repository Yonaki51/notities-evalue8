## portal
- [x] phase 0 - order information select boxes slaan op bij on blur.
- [x] phase 4 email sturen met incorrecte data zorgt voor error
- [x] phase 1 hardware website lijkt kapot te zijn. (==alleen bij al afgeronde orders?==)
- [ ] freshdesk update contact zonder iets in te vullen geeft error (validation en if-statement in de controller zetten)
- [ ] freshdesk "delete selected"  kan niet gebruikt worden. (ik ben blind. maak dit duidelijker.)
- [ ] Een maand kiezen in een agenda werkt niet
- [ ] Een bijlage toevoegen geeft een error.


  > Discuss local Caddy/Laravel proxy configuration with colleagues.
  >
  > Uploads failed because Laravel didn’t recognize the original HTTPS connection when validating signed upload URLs. Adding trusted-proxy configuration fixed this.
  >
  > Review whether to restrict TRUSTED_PROXIES=REMOTE_ADDR to Caddy specifically, or configure HTTPS between Caddy and the application server.
  >
  > Keep Livewire’s temporary upload disk set to 'local'—that fixes the separate S3 multiple-file upload issue.