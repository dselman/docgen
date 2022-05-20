## Patient Acceptance

The patient {{name}} born on {{dob as "DD/MM/YYYY"}} hereby accepts the procedure {{procedure}}.

{{#if hasAllergies}}Patient has declared the following allergies:{{else}}Patient has no declared allergies.{{/if}}

{{#olist allergies}}
   {{name}}
{{/olist}}

The cost of the procedure will be: {{% procedureCost(contract) as "K0,0.00"%}}.

Each party hereby irrevocably agrees that process may be served on it in
any manner authorized by the Laws of {{%
    let add = address ?? Address {
        state: "London",
        country: "UK"
    };
    if add.country = "UK"
        then "the county of " ++ add.state ++ ", " ++ add.country
    else 
        "the state of " ++ add.state ++ ", " ++ add.country
%}}.
