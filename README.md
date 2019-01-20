# CaptionName
Repository designed to show broken Org 9.2 Texinfo export functionality

This code exports to Texinfo under Org 9.1, but fails with an error in 9.2:

    Cannot generate node name for type: src-block

## Source of Error

The code that breaks exporting is found at: `04f35fc473`

    @@ -466,18 +466,25 @@ INFO is a plist used as a communication channel.  See

         (defun org-texinfo--get-node (datum info)
           "Return node or anchor associated to DATUM.
        -DATUM is an element or object.  INFO is a plist used as
        -a communication channel.  The function guarantees the node or
        -anchor name is unique."
        +DATUM is a headline, a radio-target or a target.  INFO is a plist
        +used as a communication channel.  The function guarantees the
        +node or anchor name is unique."
           (let ((cache (plist-get info :texinfo-node-cache)))
             (or (cdr (assq datum cache))
         	(let* ((salt 0)
         	       (basename
         		(org-texinfo--sanitize-node
        -		 (if (eq (org-element-type datum) 'headline)
        -		     (org-texinfo--sanitize-title
        -		      (org-export-get-alt-title datum info) info)
        -		   (org-export-get-reference datum info))))
        +		 (pcase (org-element-type datum)
        +		   (`headline
        +		    (org-texinfo--sanitize-title
        +		     (org-export-get-alt-title datum info) info))
        +		   (`radio-target
        +		    (org-texinfo--sanitize-title
        +		     (org-element-contents datum) info))
        +		   (`target
        +		    (org-export-data (org-element-property :value datum) info))
        +		   (type
        +		    (error "Cannot generate node name for type: %S" type)))))
         	       (name basename))
         	  ;; Ensure NAME is unique and not reserved node name "Top".
         	  (while (or (equal name "Top") (rassoc name cache))


Undoing the changes allows the export to succeed.
