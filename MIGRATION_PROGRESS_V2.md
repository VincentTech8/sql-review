## Table of Contents
- [Table of Contents](#table-of-contents)
  - [account](#account)
  - [competitors](#competitors)
  - [dashboard](#dashboard)
  - [strategy](#strategy)
---

**Legend**  
- ✅ = Done 
- ❌ = Not yet 
- ⏳ = In progress 
- N/A = Not applicable  
- **Code** = Migration Status (whether code-side migration is complete)  
- **CRUD** = basic Create/Read/Update/Delete tested  
- **Func** = methods/properties tested (e.g., `__str__`, `@property`)  
- **Rel** = relationship fields tested (RelReferenceField / ForeignKey rules)  
- **Notes** = ℹ️ for additional comments, and ⚠️**Need Comfirm** for some tricky things  

---

### account

| Model         | Code | CRUD | Func | Rel | Notes | 
|---------------|------|------|------|-----|-------|
| FreeEmail     | ✅ | ✅ | ✅ | N/A | |
| Privileges     | ✅ | ✅ | ✅ | N/A | ℹ️ auto-increment-id helper applied | 
| Notifications  | ✅ | ✅ | ✅ | N/A | ℹ️ auto-increment-id helper applied | 
| Invoices       | ✅ | ✅ | ✅ | N/A | ℹ️ auto-increment-id helper applied | 
| Product        | ✅ | ✅ | ✅ | N/A | ℹ️ **get_absolute_url()** uses `self.product_id` (business ID) instead of `self.id` to maintain integer URL format ℹ️ auto-increment-id helper applied | 
| OrderProduct   | ✅ | ✅ | ✅ | N/A | 
| Order          | ✅ | ✅ | ✅ | N/A | ℹ️ auto-increment-id helper applied | 
| PrivilegesSets | ✅ | ✅ | ✅ | N/A | | 
| CartItem       | ✅ | ✅ | ✅ | ✅ (DENY: CartItem → Product) | ℹ️ **signals.py** here only can support Forward Relationship not Reverse Relationship | 
| LineItem       | ✅ | ✅ | ✅ | ✅ (CASCADE: LineItem → Product) | |
| PurchaseCode   | ✅ | ✅ | ✅ | ✅ (DO_NOTHING: PurchaseCode → Company) | ℹ️ auto-increment-id helper applied | 
| VoucherCode    | ✅ | ✅ | ✅ | ✅(DO_NOTHING: PurchaseCode → Company) | ℹ️ auto-increment-id helper applied | 
| IntegratedTool | ✅ | ✅ | N/A | ✅(DO_NOTHING: PurchaseCode → Company) |  |
| CompanyInvoice | ✅ | ✅ | N/A | N/A |  | 
| Company        | ✅ | ✅ | ✅ | ✅(DO_NOTHING: company_owner → Users, CASCADE: notifications → Notifications) | | 
| UserDetails    | ✅ | ✅ | ✅ | N/A | | 
| Address        | ✅ | ✅ | N/A | N/A | | 
| SocialMedias   | ✅ | ✅ | N/A | N/A | | 
| UserCompany    | ✅ | ✅ | ✅ | N/A | | 
| Users          | ✅ | ✅ | ✅ | ✅(DO_NOTHING: invited_by → Users, DO_NOTHING: last_visited_company → Company, CASCADE: user_privileges → Privileges) | ⚠️ Django auth integration ⚠️ **UserDetailsForm** not integrated in contact_details field; Form validation must be manually handled in views layer when migrating ⚠️ **get_company()** method has syntax errors from original code | 

---

### competitors

| Model       | Code | CRUD | Func | Rel | Notes |
|-------------|------|------|------|-----|-------|
| Competitors | ✅ |   ✅   |   N/A   |   ✅(DO_NOTHING: Competitors → Company)   |       |

---

### dashboard

| Model             | Code | CRUD | Func | Rel | Notes |
|-------------------|------|------|------|-----|-------|
| HarvestData       |   ✅   |   ✅   |   ✅   |  N/A   |   ⚠️ **str** returns 'harvest_data', use email_id instead for now  |
| CompanyDashboard  |   ✅   |   ✅   |    ✅   |  ✅(DENY: CompanyDashboard → Company)  |ℹ️ save() auto-updates timestamp  
| CompanyReports    |   ✅   |   ✅   |    ✅   |  ✅(DENY: CompanyReports → Company)  |  ℹ️ Complex CRM integration; save() auto-updates timestamp; ⚠️ **'reporting_frequency'** field exist in current database records but not in previous Django Model, 'reporting_frequency' field created but details need comfirm, ⚠️ **'dynamics365_data'** in Django model but not in DB records
| MediaList         |   ✅   |   ✅   |    ✅   |  ✅(DENY: MediaList → Company)  |    ℹ️ str returns Company str (original Account app/Company Model primary key 'company_domain')   |
| Analysis          |   ✅   |   ✅   |    ✅   |  ✅(DENY: Analysis → Company)  |  ℹ️ str returns Company str (original Account app/Company Model primary key 'company_domain')
| QueuedOutlookEmail|   ✅   |   ✅   |    ✅   |  N/A  |  ⚠️ **str** returns a non-existent field 'analysis_company'   |  
| Generated_Content |   ✅   |   ✅   |    ✅   |  ✅(DENY: Generated_Content → Company)  |  ℹ️ str returns Company str (original Account app/Company Model primary key 'company_domain')

---

### strategy

| Model                    | Code | CRUD | Func | Rel | Notes |
|---------------------------|------|------|------|-----|-------|
| Locations                 |✅ |✅ | N/A | N/A | |
| Country                   |✅ |✅ | N/A | N/A | ⚠️ Database contains **currency** field not in original model. Temporarily set 'strict' mode to True to pass tests. |
| Industries                |✅ |✅ | N/A | N/A | |
| Campaign                  |✅ | ⏳ | ✅ save() | N/A | ⚠️ **campaign_creator** references auth_user (Django User); property `creator_user` requires Django User table; CRUD & functions tested with mock user IDs ℹ️ auto-increment-id helper applied|
| Strategy_Prefill          | ✅| ✅| N/A | N/A | ⚠️ Database contains more fields not in original model. Temporarily set 'strict' mode to True to pass tests. |
| Strategy                  |✅| ✅|✅ save() | N/A|⚠️ **strategy_creator & registered_users** reference auth_user (Django User), properties `creator_user` & `registered_user_objects` require Django User table, CRUD & functions tested with mock user IDs; ℹ️ auto-increment-id helper applied |
| Strategy_Notes            |✅ |✅| N/A| ✅(DO_NOTHING: Strategy_Notes → Strategy) | ℹ️ auto-increment-id helper applied|
| Strategy_Downloads        |✅ |✅| N/A| ✅(DO_NOTHING: Strategy_Downloads → Strategy) | ℹ️ auto-increment-id helper applied|
| SEO_result                |✅ |✅| N/A| ✅(DO_NOTHING: SEO_result  → Strategy) | ℹ️ auto-increment-id helper applied|
| Linkedin_group_client     |✅ |✅| N/A| ✅(DO_NOTHING: Linkedin_group_client → Strategy) | ℹ️ auto-increment-id helper applied|
| Linkedin_group_suggestion |✅ |✅| N/A| ✅(DO_NOTHING: Linkedin_group_suggestion → Strategy) | ℹ️ auto-increment-id helper applied|
| Brand                    |✅ |✅ | N/A | N/A | |
| Website                  |✅ |✅ | N/A | N/A | |
| MarketingObjective       |✅ |✅ | N/A | N/A | |
| BusinessProfile          |✅ |✅ | N/A | N/A |ℹ️ Used in ListField |
| ConsumerProfile          |✅ |✅ | N/A | N/A |ℹ️ Used in ListField |
| InterComm                |✅ |✅ | N/A | N/A | |
| VideoContent             |✅ |✅ | N/A | N/A | |
| ReputationMgmt           |✅ |✅ | N/A | N/A | |
| Seminars                 |✅ |✅ | N/A | N/A | |
| Youtube                  |✅ |✅ | N/A | N/A | |
| LinkedIn                 |✅ |✅ | N/A | N/A | |
| Xing                     |✅ |✅ | N/A | N/A | |
| Podcast                  |✅ |✅ | N/A | N/A | |
| Tiktok                   |✅ |✅ | N/A | N/A | |
| Twitter                  |✅ |✅ | N/A | N/A | |
| Blog                     |✅ |✅ | N/A | N/A | |
| Snapchat                 |✅ |✅ | N/A | N/A | |
| Reddit                   |✅ |✅ | N/A | N/A | |
| Ebay                     |✅ |✅ | N/A | N/A | |
| Amazon                   |✅ |✅ | N/A | N/A | |
| Etsy                     |✅ |✅ | N/A | N/A | |
| Influencers              |✅ |✅ | N/A | N/A | |
| Clubhouse                |✅ |✅ | N/A | N/A | |
| Instagram                |✅ |✅ | N/A | N/A | |
| PublicSpeaking           |✅ |✅ | N/A | N/A | |
| Publishing               |✅ |✅ | N/A | N/A | |
| Pinterest                |✅ |✅ | N/A | N/A | |
| WhatsApp                 |✅ |✅ | N/A | N/A | |
| Facebook                 |✅ |✅ | N/A | N/A | |
| CaseStudy                |✅ |✅ | N/A | N/A | |
| PR                       |✅ |✅ | N/A | N/A | |
| Advertisement            |✅ |✅ | N/A | N/A | |
| MagazineAd               |✅ |✅ | N/A | N/A | |
| NewspaperAd              |✅ |✅ | N/A | N/A | |
| BillboardAd              |✅ |✅ | N/A | N/A | |
| CinemaAd                 |✅ |✅ | N/A | N/A | |
| OutdoorAd                |✅ |✅ | N/A | N/A | |
| OnlineAd                 |✅ |✅ | N/A | N/A | |
| CommercialAd             |✅ |✅ | N/A | N/A | |
| RadioAd                  |✅ |✅ | N/A | N/A | |
| TransitAd                |✅ |✅ | N/A | N/A | |
| ExperientialAd           |✅ |✅ | N/A | N/A | |
| OTTAd                    |✅ |✅ | N/A | N/A | |
| AnnualReport             |✅ |✅ | N/A | N/A | |
| Competitor               |✅ |✅ | N/A | N/A | ℹ️ Used in ListField|
| PrintPlatform            |✅ |✅ | N/A | N/A | ℹ️ Used in ListField|
| DirectMarketing          |✅ |✅ | N/A | N/A | |
| ElectronicDirectMarketing|✅ |✅ | N/A | N/A | |
| Alliance                 |✅ |✅ | N/A | N/A | ℹ️ Used in ListField|
| Testimonial              |✅ |✅ | N/A | N/A | |
| Event                    |✅ |✅ | N/A | N/A | ℹ️ Used in ListField|
| CustomerMarketing        |✅ |✅ | N/A | N/A | |
| MarketingTradeShowsEvents|✅ |✅ | N/A | N/A | |
| Awards                   |✅ |✅ | N/A | N/A | |
| StepNote                 |✅ |✅| N/A | N/A | |



