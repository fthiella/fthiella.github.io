---
date: 2009-02-18
title: Browsing di Active Directory tramite Recordset
categories:
  - windows
---
Esistono molte utility per effettuare il browsing di Active Directory, e che permettono di esportare i risultati di una interrograzione in svariati formati, tra cui testo, CSV, Excel, etc.  
Spesso ho bisogno di fare delle elaborazioni particolarmente approfondite sui dati estratti, ed allora mi risulta molto comodo lavorare direttamente con Microsoft Access. La lettura di Active Directory tramite l'utilizzo di un Recordset pertanto è l'ideale!  

    
    Function LeggiAD()
        Dim adoConnection As New ADODB.Connection
        Dim adoCommand As New ADODB.Command
        Dim adoRecordset As New ADODB.Recordset
        Dim objRootDSE As Object
        Dim strDNSDomain As String
    
        Set adoConnection = New ADODB.Connection
        adoConnection.Provider = "ADsDSOOBject"
        adoConnection.Open ("Active Directory Provider")
        Set adoCommand.ActiveConnection = adoConnection
    
        Set objRootDSE = GetObject("<ldap://RootDSE/>">
        strDNSDomain = objRootDSE.Get("defaultNamingContext")
        adoCommand.CommandText = "<ldap://" & strDNSDomain & >;" & _
                                 "(&(objectCategory=person)" & _
                                 "(objectClass=user));" & _
                                 sAMAccountName,cn;subtree"
    
        adoCommand.Properties("Page Size") = 100
        adoCommand.Properties("Timeout") = 30
    
        adoCommand.Properties("Cache Results") = False
        Set adoRecordset = adoCommand.Execute
        Do Until adoRecordset.EOF
            rem fai qualcosa con adoRecordset("sAMAccountName").Value
            rem fai qualcosa con adoRecordset("cn").Value
    
            adoRecordset.MoveNext
        Loop
        adoRecordset.Close
        adoConnection.Close
    
        Set adoConnection = Nothing
        Set adoCommand = Nothing
        Set objRootDSE = Nothing
        Set adoRecordset = Nothing
    End Function
    
Questo codice scorre tutti gli utenti presenti in Active Directory (filtrati da objectCategory=person e da objectClass=user) ed estrae i campi sAMAccountName e cn: i dati restituiti potrebbero poi essere inseriti in un altra tabella, o potrebbero essere elaborati in altro modo. In genere utilizzo questo codice come base di partenza, e lo personalizzo a seconda delle necessità!
