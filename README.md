# Add-in
Tool SQL + Visual
https://drive.google.com/drive/folders/1wdYiCDYxR7ErrQhTrWT2Qx1JEwUqEn5S?usp=sharing
=======================================
System.Web.UI.ICallbackEventHandler

#region Đăng ký ICallback
  String cbReference = this.Page.ClientScript.GetCallbackEventReference(this, "arg", "ReceiverDiagram", "context");
  String callbackScript = "function CallDiagram(arg, context)" + "{ " + cbReference + ";}";
  Page.ClientScript.RegisterClientScriptBlock(this.GetType(), "Diagram", callbackScript, true);
#endregion

#region ICallback
        public string GetCallbackResult()
        {
            return StringResult;
        }

        public void RaiseCallbackEvent(string eventArgument)
        {
            string strCaseName = string.Empty;
            try
            {
                string[] arrValue = eventArgument.Split(new string[] { "##$$" }, StringSplitOptions.None);
                switch (arrValue[0])
                {
                    #region DeleteWK
                    case "DeleteWK":
                        {
                            strCaseName = "DeleteWK";
                            string StringStepId = arrValue[1];
                            string StringWorkflowId = this.Page.Request.QueryString["WID"] + string.Empty;

                            SPWeb currentWeb = SPContext.Current.Site.RootWeb;

                            currentWeb.AllowUnsafeUpdates = true;
                            SPList ListWorkflowStep = currentWeb.Lists[Pro_ListWorkflowStepName];

                            SPListItem ItemWorkflowStep = ListWorkflowStep.GetItemById(Convert.ToInt32(StringStepId));


                            SPFieldLookupValue lkWorkflow = new SPFieldLookupValue(ItemWorkflowStep["Workflow"] + string.Empty);
                            string NextStep = ItemWorkflowStep["NextStep"] + string.Empty;
                            string CurrentStep = ItemWorkflowStep["Step"] + string.Empty;

                            SPList ListWorkflowDefine = SPContext.Current.Web.Lists["WorkflowDefine"];
                            SPListItem ItemWorkflowDefine = ListWorkflowDefine.GetItemById(lkWorkflow.LookupId);

                            if (CheckWorkflowPending(int.Parse(CurrentStep), ItemWorkflowDefine))
                            {
                                StringResult = "DeleteWKError##$$" + (IntLCID == 1066 ? pro_strMessError.Split('|')[0] : pro_strMessError.Split('|')[1]);
                                break;
                            }

                            SPQuery query = new SPQuery();
                            query.Query = "<Where><And><Geq><FieldRef Name='Step' /><Value Type='Number'>" + CurrentStep + "</Value></Geq><Eq><FieldRef Name='Workflow' LookupId='true' /><Value Type='Lookup'>" + lkWorkflow.LookupId + "</Value></Eq></And></Where><OrderBy><FieldRef Name='Step' Ascending='True' /></OrderBy>";
                            SPListItemCollection ItemStepsColl = ListWorkflowStep.GetItems(query);

                            // Là bước cuối cùng thì cập nhật lại bước trước nó
                            if (ItemStepsColl.Count == 1)
                            {
                                query = new SPQuery();
                                query.Query = "<Where><And><Eq><FieldRef Name='NextStep' /><Value Type='Number'>" + ItemStepsColl[0]["Step"] + "</Value></Eq><Eq><FieldRef Name='Workflow' LookupId='true' /><Value Type='Lookup'>" + lkWorkflow.LookupId + "</Value></Eq></And></Where>";
                                SPListItemCollection ItemPreStepsColl = ListWorkflowStep.GetItems(query);
                                if (ItemPreStepsColl.Count > 0)
                                {
                                    SPListItem ItemPreStep = ItemPreStepsColl[0];
                                    ItemPreStep["NextStep"] = null;
                                    ItemPreStep.Update();

                                    _CoreWorkflowWeb.Update_WorkflowStepDefine("", ItemPreStep);
                                }
                            }

                            ResetSteps(ItemStepsColl, ItemWorkflowStep.ID);

                            ItemWorkflowStep.Delete();
                            //Delete SQL
                            _CoreWorkflowWeb.Delete_WorkflowStepDefine("", StringStepId);

                            UpdateWorkflowItems(int.Parse(CurrentStep), ItemWorkflowDefine);

                            currentWeb.AllowUnsafeUpdates = false;

                            StringResult = "DeleteWK##$$";
                            break;
                        }
                        #endregion


                }
            }
            catch (Exception ex)
            {
                SPListUtilities.TrackingError("Author: Toannv - Method: RaiseCallbackEvents(strCaseName: " + strCaseName + ") - WP: WorkflowDiagram - Project: VuThao.WorkflowFormDefination - User: " + SPContext.Current.Web.CurrentUser.Name, ex.ToString());
            }
        }
#endregion


    function DeleteStep(stringStepId, selectedElement) {
        if (confirm("<%= (System.Threading.Thread.CurrentThread.CurrentUICulture.LCID == 1066 ? "Bạn có chắc muốn xóa nó?" : "Are you want to remove it?")%>")) {
            var tdElement = selectedElement.parentNode.parentNode;
            var trElement = tdElement.parentNode;
            var tbElement = trElement.parentNode;

            CallDiagram("DeleteWK##$$" + stringStepId, "");
            ShowLoading();
        }
    }
=======================================
https://app.prntscr.com/en/download.html
+++++++++++++++++++++++++
https://drive.google.com/drive/u/0/folders/0B0Nnh1WLx7MuZzNKUU1leF9meWM
+++++++++++++++
C:\Program Files\Common Files\microsoft shared\Web Server Extensions\16\LOGS
=====================


https://collab365.community/forum/topics/forcing-ie-to-open-workflow-task-related-content-link-in-browser/
