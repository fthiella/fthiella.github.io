---
date: 2009-07-28
---
# Mappare Condivisioni di Rete in base al Gruppo di appartenenza

Molte volte nello script di logon c'è la necessità di mappare condivisioni di rete in base ai gruppi cui appartiene un utente.  
L'articolo [How Can I Map Drives Based on Membership in a Group?](<http://www.microsoft.com/technet/scriptcenter/resources/qanda/dec04/hey1210.mspx>) tratto dal sito di Microsoft spiega molto bene il problema e fornisce una valida introduzione; lo script presentato però è molto semplificato. Ho cercato di estenderlo in base alle esigenze della mia azienda.  
  
Gruppo Primario  
Il codice tratto dall'articolo di Microsoft restituisce tutti i gruppi, **ad esclusione del gruppo primario dell'utente** :  
  

    
    Set objSysInfo = CreateObject("ADSystemInfo")
    Set objNetwork = CreateObject("Wscript.Network")
    
    strUserPath = "LDAP://" & objSysInfo.UserName
    Set objUser = GetObject(strUserPath)
    
    For Each strGroup in objUser.MemberOf
      strGroupPath = "LDAP://" & strGroup
      Set objGroup = GetObject(strGroupPath)
      strGroupName = objGroup.CN
      Wscript.Echo strGroupName 
    Next
    
Se la objectUser.MemberOf contiene soltanto un gruppo questo il codice qui sopra va in errore, poiché MemberOf non è un array. Bisogna quindi distinguere se si tratta di un array oppure di un gruppo singolo in questo modo:  

    
    If isArray(objUser.MemberOf) then
      For Each strGroup in objUser.MemberOf
          strGroupPath = "LDAP://" & strGroup
          Set objGroup = GetObject(strGroupPath)
          checkGroup Unit, objGroup.CN
      Next
    Else
        strGroupPath = "LDAP://" & objUser.MemberOf
        Set objGroup = GetObject(strGroupPath)
        checkGroup Unit, objGroup.CN
    End If
    
  
Alla collezione MemberOf manca dunque il gruppo primario dell'utente, che generalmente è _Domain Users_ ma potrebbe sempre essere stato modificato. Possiamo però ottenere il codice del gruppo primario con _objUser.PrimaryGroupID_ e scorrere l'elenco dei gruppi fino a che non troviamo il gruppo con questo ID. La funzione che restituisce il nome del gruppo dato l'ID è la seguente:  
  

    
    Function GetPrimaryGroup(ByVal GroupID)
    Set objConnection = CreateObject("ADODB.Connection")
    Set objCommand = CreateObject("ADODB.Command")
    
    objConnection.Provider = "ADsDSOObject"
    objConnection.Open "Active Directory Provider"
    
    Set objCommand.ActiveConnection = objConnection
    objCommand.Properties("Page Size") = 100
    objCommand.Properties("Timeout") = 30
    objCommand.Properties("Cache Results") = False
    
    Set objRootDSE = GetObject("LDAP://RootDSE")
    strDNSDomain = objRootDSE.Get("defaultNamingContext")
    
    objCommand.CommandText = "<ldap://" & strdnsdomain & ">;" &_
      "(objectClass=group);" &_
      "sAMAccountName,primaryGroupToken;subtree"
    
    Set objRecordSet = objCommand.Execute
    
    Do Until objRecordSet.Eof
      If objRecordSet("primaryGroupToken").Value = GroupID Then
        GetPrimaryGroup = objRecordSet("sAMAccountName").Value
        Exit Do
      End If
      objRecordSet.MoveNext
    Loop
    
    objRecordset.Close
    objConnection.Close
    
    Set objRecordset = Nothing
    Set objConnection = Nothing
    End Function
    
  
Non sono soddisfatto di questa funzione, in quanto non sono riuscito a filtrare il gruppo direttamente nel CommandText e sono pertanto costretto a scorrere tutti i gruppi tramite Loop, fino a che non trovo il gruppo con il codice ID che stavo ricercando, ma.... comunque funziona.  
  
Associazione Gruppi-Condivisioni  
Tramite l'oggetto Scripting.Dictionary creo un elenco di associazioni tra gruppi e relative aree condivise, come ad esempio:  
  

    
    Set d = CreateObject("Scripting.Dictionary")
    
    d.Add "Finanziario", "\\server01\fs\Finanziario"
    d.Add "Personale", "\\server01\fs\Personale"
    d.Add "Legale", "\\server02\Legale"
    
  
MapNetworkDrive  
A questo punto posso scorrere tutto l'elenco dei gruppi cui l'utente appartiene, passando ogni singolo gruppo alla seguente procedura. Se il gruppo appartiene alla lista dei gruppi a cui è associata un'area condivisa, tale area viene montata nel disco "Unita".  
  

    
    Sub checkGroup(ByRef Unita, ByVal Group)
    On Error Resume Next
      If d.Exists(Group) Then
        objNetwork.RemoveNetworkDrive Unita & ":"
        objNetwork.MapNetworkDrive Unita & ":", d(Group), False
        Unita = Chr(Asc(Unita) + 1)
      End If
    End Sub
    
  
Codice Finale  
Il codice finale è il seguente, a cui vanno aggiunte la sub e la funzione presentate in precedenza:  
  

    
    On Error Resume Next
    
    Set objSysInfo = CreateObject("ADSystemInfo")
    Set objNetwork = CreateObject("Wscript.Network")
    
    rem *** associazioni gruppo-percorso
    Set d = CreateObject("Scripting.Dictionary")
    
    d.Add "Finanziario", "\\server01\fs\Finanziario"
    d.Add "Personale", "\\server01\fs\Personale"
    d.Add "Legale", "\\server02\Legale"
    
    rem *** prima unità di rete da connettere
    Unit = "O"
    
    rem *** utente corrente
    Set objUser = GetObject("LDAP://" & objSysInfo.UserName)
    
    rem *** controlla il gruppo primario ***
    checkGroup Unit, GetPrimaryGroup(objUser.PrimaryGroupID)
    
    rem *** controlla gli altri gruppi ***
    For Each strGroup in objUser.MemberOf
    strGroupPath = "LDAP://" & strGroup
    Set objGroup = GetObject(strGroupPath)
    checkGroup Unit, objGroup.CN
    Next
    
    rem *** cleanup
    d.RemoveAll
    
  
Se un utente appartiene a più gruppi che dispongono di un'area associata, questo script monta tutti i percorsi di rete a partire dalla lettera O:, poi P:, Q: etc...  
L'area di lavoro associata al gruppo primario verrà sempre montata come unità O:, mentre le altre aree saranno montate a seguire.  
Se la stessa area di lavoro è associata a più gruppi, ed un utente appartiene a più di uno di questi gruppi, la stessa area verrà montata più volte.  
  
Happy logging on!
