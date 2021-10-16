require 'nokogiri'

desc "Use nokogiri to generate HTML pages from TEI xml"
task :letters, [:arg1] do |t, args|
    args.with_defaults(
        :arg1 => "*/*"
      )

    # iterate over all XML files in input dir
    Dir.glob("xml/#{args.arg1}.xml").each do |xmlname|
        names = xmlname.split("/")
        filename = names[2]
        lettername = filename.split(".").first

        # Get output name
        output_name = lettername + ".html"

        # read the xml with Nokogiri
        doc = Nokogiri::XML(File.read(xmlname))

        # open new file
        newdoc = File.new("_letters/" + output_name, 'w')

        # frontmatter
        def frontmatter(letter, number, auth, rec, date, locations, people, languages, title, images, thumbnail, full, manifest)
                "---\nletter_id: " + letter + "\nnumber: " + number + "\ncreator: " + auth + "\nrecipient: " + rec + "\nletter_date: " + date + "\nlocations: " + locations + "\npeople: " + people + "\nlanguage: " + languages + "\nlayout: letter" + "\ntitle: " + title + "\nimages: " + images + "\nthumbnail: " + thumbnail + "\nfull: " + full + "\nmanifest: " + manifest + "\n---\n\n"
        end

        key = doc.css('teiHeader fileDesc sourceDesc bibl title').attr("key")
        doc.css('teiHeader fileDesc sourceDesc bibl title').attr("n").nil? ? n = "" : n = doc.css('teiHeader fileDesc sourceDesc bibl title').attr("n")
        creator = doc.css('teiHeader fileDesc sourceDesc bibl author').text.strip
        recipient = doc.css('teiHeader fileDesc sourceDesc bibl persName').text.strip
        date = doc.css('teiHeader fileDesc sourceDesc bibl date').attr("when")
        title = creator + " to " + recipient + ", " + date
        firstimage = doc.at_css('text body div[1]').attr("facs")
        images = doc.css('teiHeader fileDesc sourceDesc msDesc msContents').attr("facs")
        thumbnail = '/img/derivatives/iiif/images/' + key + '_' + firstimage + '/full/250,/0/default.jpg'
        full = '/img/derivatives/iiif/images/' + key + '_' + firstimage + '/full/1140,/0/default.jpg'
        manifest = '/img/derivatives/iiif/' + key + '/manifest.json'

        # get unique locations keys
        letter_locations = doc.css('text body div[3] placeName')
        locations_st = letter_locations.map {|element| element["key"]}.join(';')
        locations_ar = locations_st.split(';').uniq
        locations_uniq = locations_ar.join('; ')

        # get unique person keys
        letter_people = doc.css('text body div[3] persName')
        people_st = letter_people.map {|element| element["key"]}.join(';')
        people_ar = people_st.split(';').uniq
        people_uniq = people_ar.join('; ')

        # get unique language keys
        letter_languages = doc.css('text body div head')
        languages_st = letter_languages.map {|element| element["xml:lang"]}.join(';')
        languages_ar = languages_st.split(';').uniq
        languages_ar.map! do |node|
            if node == 'en'
                node = 'English'
            elsif node == 'it'
                node = 'Italian'
            elsif node == 'fr'
                node = 'French'
            else node == 'sp'
                node = 'Spanish'
            end
        end
        languages_uniq = languages_ar.join('; ')

        imageset = doc.css('text body div[1] pb') # get all pb elements in div
        otherimage = imageset.map {|element| element["facs"]}.join(';') # => output as array: ["name key 1"; "name key 2"] => name key 1; name key 2
        
        # get all elements with attribute "style"
            # id_set = doc.xpath("//*[@style]")
        # put ids in string, each separated by ';':
            # all_ids = id_set.map {|element| element["style"]}.join('; ')

        # remove all facs divs
        # doc.xpath('//div[@facs=""]').remove

        # add frontmatter to newdoc first
        newdoc << frontmatter(key, n, creator, recipient, date, locations_uniq, people_uniq, languages_uniq, title, images, thumbnail, full, manifest)

        people_uniq.each do |node|
            if node == doc.at_css('persName').attr('key')
                node.name = 'a'
                node['class'] = 'pop-annotation'
                node['tabindex'] = '0'
                node['id']

        # hyperlink persNames
        doc.css('persName').each do |node|
            node.name = 'a'
            key = node['key']
            node['href'] = '/mancini_source/people.html#' + key
        end

        # hyperlink placeNames
        doc.css('placeName').each do |node|
            node.name = 'a'
            key = node['key']
            node['href'] = '/mancini_source/places.html#' + key
            node.delete('key')
        end

        # do annotations
        doc.css('p').children.each do |node|
            if node['type']
            text = node['style']
            node.name = 'a'
            id = node['xml:id']
            node['href'] = '#' + id
            node['data-toggle'] = 'tooltip'
            node['title'] = text
            original_node = node.content
                if node['key']
                    key = node['key']
                    node.before '<a href="/mancini_source/people.html#' + key + '">' + node.content + '</a> '
                end
            node.content = '[' + node['n'] + ']'
            node.delete('n')
            node.delete('style')
            node.delete('key')
            node.delete('type')
            end
        end

         # text emphasis
        doc.css('hi').each do |node|
            rend = node['rend']
            node.name = rend
            node.delete('rend')
        end

        # remove unused nodes, teiHeader and note
        doc.css('TEI teiHeader').remove
        doc.css('note').remove
        
        # remove xml namespace
        doc.remove_namespaces!

        # change div type to tabpanel div

        doc.css('TEI text body div').each do |node|
            type = node.attr('id').to_s
            if type.include? "trans"
                node['class'] = 'tab-pane fade show active'
                node['id'] = 'trans'
                node['role'] = 'tabpanel'
                node['aria-labelledby'] = 'trans-tab'
                node.delete('type')
                node.delete('n')
                # delete facs (temporary?)
                node.delete('facs')
            elsif type.include? "edit"
                node['class'] = 'tab-pane fade'
                node['id'] = 'edit'
                node['role'] = 'tabpanel'
                node['aria-labelledby'] = 'edit-tab'
                node.delete('type')
                node.delete('n')
                # delete facs (temporary?)
                node.delete('facs')
            else type.include? "raw"
                node['class'] = 'tab-pane fade'
                node['id'] = 'raw'
                node['role'] = 'tabpanel'
                node['aria-labelledby'] = 'raw-tab'
                node.delete('type')
                node.delete('n')
                # delete facs (temporary?)
                node.delete('facs')
            end
        end

        # change head to heading
        doc.css('head').each do |node|
            new_node = doc.create_element 'h4'
            type = node['type'].to_s.capitalize
            language = node['lang'].to_s
            case language
            when "en"
                full_lang = "English"
            when "it"
                full_lang = "Italian"
            when  "fr"
                full_lang = "French"
            when "sp"
                full_lang = "Spanish"
            end
            new_node.inner_html = type + " (" + full_lang + ")"
            node.replace new_node
            new_node['class'] = 'mb-3'
        end

        # remove pb (temporary?)
        doc.css('pb').remove

        # define paragraph method
        def paragraph(doc, element)
            element.each do |node|
                new_node = doc.create_element 'p'
                new_node.inner_html = node.inner_html
                node.replace new_node
            end
        end

        # define node removal method
        def node_removal(node_removed)
            node_removed.each do |node|
                node_content = node.inner_html
                node.before node_content
                node.remove
            end
        end

        # dateline element to paragraph
        dateline = doc.css('dateline')
        paragraph(doc, dateline)

        # date element to text
        date_opener = doc.css('opener date')
        node_removal(date_opener) unless date_opener.attr('type')

        # remove opener element
        opener = doc.css('opener')
        node_removal(opener)

        # salute and signed elements to paragraph
        salute = doc.css('salute')
        signed = doc.css('signed')
        paragraph(doc, salute)
        paragraph(doc, signed)

        doc.css('closer p').each do |node|
            if node['class']
                node['class'] = 'text-right'
            end
        end
        
        # remove closer element
        closer = doc.css('closer')
        node_removal(closer)

        # add remaining content, remove extra blank lines
        # find a way to pretty print?

        doc.css('TEI text div').each do |x|
            newdoc << x.to_s.gsub(/^\s*\n/, "") + "\n"
        end

        # close file
        newdoc.close
    end
end

# Deploy site
desc "Build site with production env"
task :deploy do
  ENV["JEKYLL_ENV"] = "production"
  sh "jekyll build"
end