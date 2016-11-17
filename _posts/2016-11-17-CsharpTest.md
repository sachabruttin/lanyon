---
layout: post
title: C# Test
---

### Hello C'##


    protected override async void OnEntityChanged()
    {
      base.OnEntityChanged();
         
      if (Entity != null)
      {
        await LoadObject();
        await LoadTakedOutItem();
        this.RaisePropertyChanged(x => x.BankingAccount);
        this.RaisePropertyChanged(x => x.CanWithdrawDocuments);
        this.RaisePropertyChanged(x => x.IsCoverCheckedOut);
        this.RaisePropertyChanged(x => x.IsObjectLoaded);
        this.RaisePropertyChanged(x => x.IsIssuePrintingEnabled);
        this.RaisePropertyChanged(x => x.IsBlueFolderPrintingEnabled);
              
        Entity.PropertyChanged += async (s, e) =>
            {
              if (e.PropertyName == "NocSufix")
              {
                await LoadObject();
                await LoadTakedOutItem();
                this.RaisePropertyChanged(x => x.BankingAccount);
                this.RaisePropertyChanged(x => x.IsObjectLoaded);
                this.RaisePropertyChanged(x => x.IsIssuePrintingEnabled);
                this.RaisePropertyChanged(x => x.IsBlueFolderPrintingEnabled);
              }
                      
              if (e.PropertyName == "IsCoverCheckedOut")
              {
                this.RaisePropertyChanged(x => x.IsCoverCheckedOut);       
              }
                      
              this.RaisePropertyChanged(x => x.CanWithdrawDocuments);
             };
        }
    }

